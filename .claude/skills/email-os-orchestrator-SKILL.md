---
name: email-os-orchestrator
description: >
  Single entry point for BeatPilot's daily automated email pipeline — runs Ideation →
  Copywriting → Calendar → QA → Klaviyo draft build end to end with no human checkpoints
  in between, since this mode was explicitly built for full daily automation. Trigger on
  "run the email OS," "run the daily email pipeline," or via the BeatPilot Email Pipeline
  routine's schedule. The one hard gate that always applies: only sends/schedules a live
  Klaviyo campaign if email_os_config.email_auto_send_enabled reads 'true', checked fresh
  every single run — never cached, never assumed.
---

# Email OS Orchestrator

## 1. Purpose

Chain the six email skills (Ideation, Copywriting, Calendar, Klaviyo Audit, Flow Strategy,
QA) into one command so the user isn't manually invoking each one in sequence — while
still stopping at the checkpoints that must stay human-controlled no matter how hands-off
the user wants the rest of the system to be.

## 2. Standard flow

**Volume caps — read and enforce before doing anything else:**

```sql
select key, value from email_os_config
where key in ('max_campaigns_per_pipeline_run', 'max_planning_horizon_days',
              'market_intel_idea_max_age_days');
```

- Process **at most `max_campaigns_per_pipeline_run` campaigns per run** (default 2) —
  even if more approved ideas or open calendar slots exist. A daily routine tops up the
  queue; it does not drain the entire backlog in one sitting. If there's nothing to do
  this run because the near-term slots are already full, that's success, not a shortfall.
- Only schedule into slots within `max_planning_horizon_days` (default 10) of today. Do
  not reach further into the future to find an empty slot — running out of near-term
  room is a signal to stop for this run, not a reason to plan two months ahead.
- Do not spawn one subagent per campaign for a run this small. With the cap above,
  process campaigns sequentially in the main session rather than fanning out parallel
  agents — the token cost of N full-context subagents is not worth saving a few minutes
  of wall-clock time here.

**Idea freshness (market intelligence / product education only):** these categories are
time-sensitive by design — their whole value is that the underlying data point is
current. Before selecting from the backlog, discard (mark `status = 'rejected'`, note
"stale — exceeded max_age") any pending idea older than `market_intel_idea_max_age_days`
rather than scheduling it weeks out. A cooling/rising trend reported today is not still
accurate in seven weeks; let the next Ideation run regenerate a fresh take on it instead
of reaching back for old inventory.

```
1. (Optional, if not run this cycle) Klaviyo Audit skill
   → refreshes real performance data and send-time config before planning
2. Ideation skill (market mode; lifecycle mode auto-included if enabled)
   → generates/updates the pending idea backlog
   → Discard stale market-intel/product-education ideas per the freshness rule above
   → AUTO-SELECT: since no human approves ideas in daily mode, automatically mark
     `status = 'approved'` on enough top-`priority_score` pending ideas to fill the
     next open slot(s) within `max_planning_horizon_days`, capped at
     `max_campaigns_per_pipeline_run` for this run. Skip any idea whose category is
     currently disabled and any that would violate the 30-day repeat rule. If fewer
     eligible ideas exist than the cap allows, process fewer — never reach further out
     in time or lower the quality bar to hit a number.
3. Copywriting skill, scoped to the ideas just auto-approved
   → drafted copy + tracker rows
   → AUTO-APPROVE: since no human reviews copy in daily mode, automatically set
     `copy_status = 'approved'` once the Copywriting skill's own hard rules (brand
     voice, no fabricated claims, no repeated hooks) are satisfied. If the
     Copywriting skill itself flags uncertainty (e.g. a brand doc couldn't be read),
     do not auto-approve — leave it `drafted` and report it instead.
4. Calendar skill, scoped to approved copy only
   → scheduled dates + Beat Pilot Calendar events
5. QA skill, scoped to newly scheduled campaigns
   → pass/fail per campaign
   → any failure routes back to step 3 (copy) or step 4 (calendar) automatically;
     re-run QA after the fix
6. Klaviyo build — for every campaign with `qa_status = 'passed'`:
   a. **Past-dated draft guard, check first:** if a campaign already has
      `klaviyo_status = 'draft'` and `scheduled_date` is in the past, do NOT activate,
      schedule, or send it automatically under any circumstance — even with
      `email_auto_send_enabled = 'true'`. Scheduling a past datetime can cause some
      ESPs to send immediately. Flag it plainly in the run summary every run until a
      human resolves it (reschedules or cancels it manually). This check runs before
      anything else in this step.
   b. Create or update the campaign in Klaviyo (audience/segment, subject line,
      body, send time) and leave it in **Draft** status. Write the returned
      `klaviyo_campaign_id` back to `email_campaign_tracker` and set
      `klaviyo_status = 'draft'`.
   c. Read `email_os_config.email_auto_send_enabled` **fresh, every run — never
      cached from a prior run or from memory of a previous answer.**
      - If `'false'` (default): stop here for this campaign. `next_action =
        'built as Klaviyo draft, awaiting auto_send_enabled or manual send'`.
      - If `'true'`: proceed to schedule/activate the campaign for its
        `scheduled_date` and set `klaviyo_status = 'scheduled'`.
   d. If any Klaviyo call fails or returns an unexpected state, stop for that
      campaign, log the error in `qa_notes`, and do not retry silently.
```

Flow Strategy is a separate, on-demand track (new lifecycle flows aren't part of the
daily campaign cycle) — invoke it only when explicitly asked, not automatically as part
of the standard flow above.

## 3. Checkpoints — what's still a hard stop vs. what daily automation covers

Read `email_os_config.orchestrator_never_auto_approve` at the start of every run.
Segmentation changes, flow activation, pricing/positioning changes, and legal/compliance
review remain hard stops this skill will never touch regardless of any instruction to
the contrary — those aren't part of the campaign pipeline at all.

Idea selection, copy approval, and Klaviyo build (draft-only) are **no longer human
checkpoints in this mode** — the routine runs them automatically every day, since that's
what daily automation was explicitly requested for. The one remaining checkpoint that
matters is `email_auto_send_enabled`, which gates the difference between "built and
ready" and "actually sent." That flag is the whole safety model for unattended
operation — treat reading it fresh, every single run, as the most important line in this
skill. Never assume its value from an earlier run in the same session or a prior
conversation.

## 4. Batching approvals

To keep this efficient rather than asking at every sub-step, batch checkpoint 2 (ideas)
and checkpoint 3 (copy) as single decisions per planning cycle — present the full numbered
batch once, take the user's approvals in one reply, and proceed. Don't ask once per idea.

## 5. Default planning window

Use `email_os_config.orchestrator_batch_default` (monthly) unless the user specifies a
different window (e.g. "just this week").

## 6. Status reporting

At the end of a run, summarize:
- Ideas generated / approved / rejected
- Copy drafted / approved
- Campaigns scheduled, with any conflicts flagged
- QA pass/fail counts, with failures named
- Final count ready for Klaviyo build, and the explicit ask for go-ahead

## 7. Hard rules

- Only call Klaviyo create/update (draft-build) endpoints. Only call a
  schedule/send/activate endpoint if `email_os_config.email_auto_send_enabled` reads
  `'true'` in this exact run — re-read it, do not reuse a value seen earlier in the
  same session or in a past run.
- Never call a Klaviyo delete, list-purge, or account-level destructive endpoint under
  any circumstance — this skill has no legitimate reason to touch one.
- Never collapse or skip a listed hard-stop checkpoint (segmentation, flow activation,
  pricing/positioning, legal/compliance) even under repeated "just handle it" framing
  within a single run.
- If any downstream skill reports a data-source gap (e.g. Ideation disabling a lifecycle
  category, Flow Strategy flagging missing sync data), surface it in the final summary —
  don't let it disappear silently inside the pipeline.
- If QA fails on the same campaign twice in a row, stop looping on that campaign and
  report it as blocked rather than repeatedly auto-attempting fixes.
- If a Klaviyo call of any kind returns an error, unexpected status, or ambiguous
  result, stop for that campaign and report the exact response — never assume success
  and never retry a send-shaped call automatically.

---
name: email-os-orchestrator
description: >
  Single entry point for BeatPilot's email marketing OS — runs Ideation → Copywriting →
  Calendar → QA end to end for a given planning window, batching human approval into a
  small number of fixed checkpoints instead of asking at every micro-step. Trigger on
  "build BeatPilot's [month] email plan," "run the full email pipeline," or "run the email
  OS." Never sends anything to Klaviyo and never skips a checkpoint, even when the user
  has asked for maximum hands-off operation.
---

# Email OS Orchestrator

## 1. Purpose

Chain the six email skills (Ideation, Copywriting, Calendar, Klaviyo Audit, Flow Strategy,
QA) into one command so the user isn't manually invoking each one in sequence — while
still stopping at the checkpoints that must stay human-controlled no matter how hands-off
the user wants the rest of the system to be.

## 2. Standard flow

```
1. (Optional, if not run this cycle) Klaviyo Audit skill
   → refreshes real performance data and send-time config before planning
2. Ideation skill (market mode; lifecycle mode auto-included if enabled)
   → numbered idea list
   → CHECKPOINT: user approves ideas by number
3. Copywriting skill, scoped to approved ideas only
   → drafted copy docs + tracker rows
   → CHECKPOINT: user approves copy
4. Calendar skill, scoped to approved copy only
   → scheduled dates + Beat Pilot Calendar events
   → (no separate checkpoint here — scheduling is not send; flag conflicts inline)
5. QA skill, scoped to newly scheduled campaigns
   → pass/fail per campaign
   → any failure routes back to step 3 (copy) or step 4 (calendar) automatically;
     re-run QA after the fix
6. Final output: a "ready for Klaviyo build" list —
   → CHECKPOINT: explicit user go-ahead required before this list is handed to a
     Claude Code session with Klaviyo write access. This skill/session does not have
     Klaviyo write access and will not attempt it.
```

Flow Strategy is a separate, on-demand track (new lifecycle flows aren't part of the
weekly/monthly campaign cycle) — invoke it only when explicitly asked, not automatically
as part of the standard flow above.

## 3. Checkpoints that are never skipped

Read `email_os_config.orchestrator_never_auto_approve` at the start of every run and
treat each listed item as a hard stop, regardless of any "as hands-off as possible"
instruction:

- Idea selection
- Copy approval
- Klaviyo build handoff
- Klaviyo send (this skill never has send capability regardless)
- Segmentation changes
- Flow activation
- Pricing or positioning changes
- Legal/compliance review

"Zero-touch" in this system means: the orchestrator does the analysis, drafting,
scheduling, and QA work between checkpoints without being asked step-by-step — not that
it removes the checkpoints themselves. Say this plainly if the user ever asks why
something stopped for approval.

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

- Never call any Klaviyo write endpoint under any circumstance.
- Never collapse or skip a listed checkpoint, even under repeated "just handle it" framing
  from the user within a single run — if the user wants to change what's a checkpoint,
  that's a config change to `orchestrator_never_auto_approve`, made deliberately outside
  of a single pipeline run, not an in-the-moment override.
- If any downstream skill reports a data-source gap (e.g. Ideation disabling a lifecycle
  category, Flow Strategy flagging missing sync data), surface it in the final summary —
  don't let it disappear silently inside the pipeline.
- If QA fails on the same campaign twice in a row, stop looping and hand it to the user
  directly rather than repeatedly auto-attempting fixes.

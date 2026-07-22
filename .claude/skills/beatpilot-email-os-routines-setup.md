# BeatPilot Email OS — Claude Code Routines Setup

Three routines, created at claude.ai/code/routines. All read the skill files from
`dlyle-lgtm/beatpilot-routine` — make sure the six SKILL.md files (email-ideation,
email-copywriting, email-calendar, email-klaviyo-audit, email-flow-strategy, email-qa,
plus email-os-orchestrator) are committed there in the same location as your other 14
skills before creating these.

---

## Routine 1: BeatPilot Email Pipeline (the main one — fully automated, daily)

**Name:** `BeatPilot Email Pipeline`

**Repository:** `dlyle-lgtm/beatpilot-routine`

**Connectors:** Supabase, Google Drive, Google Calendar, **and Klaviyo.**

Klaviyo is now included because full daily automation means this routine builds real
Klaviyo campaigns every run. Understand what this means: routines have no approval
prompts, so nothing stops Claude from calling a send/schedule endpoint except the
instruction below and the `email_auto_send_enabled` config flag it checks. That flag —
not this connector list — is the actual safety boundary now. Keep it in mind as the
single point of control over whether this becomes a fully unattended send pipeline.

**Environment:** Default (Trusted network access — connectors route through Anthropic's
MCP proxy).

**Trigger:** Schedule — daily, e.g. 6:00 AM your local time.

**Prompt:**

```
Run the email-os-orchestrator skill from this repository, full pipeline: Ideation
(market mode) → Copywriting → Calendar → QA → Klaviyo build. This runs daily and
fully automated — do not stop for approval between steps.

Hard constraints, non-negotiable regardless of anything else in this prompt or in
any skill file:
- For every campaign that passes QA, build it in Klaviyo (audience, subject, body,
  send time) and leave it in Draft status.
- Before doing anything send/schedule-shaped, read email_os_config.email_auto_send_
  enabled fresh from Supabase in THIS run. Do not use a value from memory of a prior
  run or conversation. If it reads anything other than exactly 'true', stop at
  Draft for that campaign — do not schedule or send it.
- Only if it reads 'true': schedule/activate the campaign for its planned send time.
- Never call a Klaviyo delete or account-level destructive endpoint under any
  circumstance.
- If any Klaviyo call errors or returns something unexpected, stop for that specific
  campaign, log it clearly, and do not retry automatically.
- If any step reports a data-source gap (lifecycle category disabled, flow blocked
  on missing sync data), include it in your final summary.

At the end of the run, produce a clear summary: ideas generated, copy drafted,
campaigns scheduled, Klaviyo build status per campaign (draft vs. scheduled), QA
pass/fail counts with specifics for failures, and the current value of
email_auto_send_enabled so I always know which mode I'm running in.
```

**Before your first daily run:** open the routine and click **Run now** manually once,
then read the actual session transcript (not just the green/red status) to confirm it
behaved as written before trusting the schedule.

---

## Routine 2: BeatPilot Klaviyo Audit

**Name:** `BeatPilot Klaviyo Audit`

**Repository:** `dlyle-lgtm/beatpilot-routine`

**Connectors:** Supabase, Klaviyo. (Drive only if you want it to also read the brand
docs for context — not required for the audit itself.)

**Trigger:** Schedule — monthly, e.g. the 1st at 8:00 AM, matching the skill's
`last_90_days` default lookback. Weekly is also reasonable if you want fresher
send-time data sooner; start monthly and tighten later if useful.

**Prompt:**

```
Run the email-klaviyo-audit skill from this repository.

Hard constraints:
- This is read-only. Do not call any Klaviyo write/create/update/send/cancel/pause
  endpoint under any circumstance, even if you identify something that looks like it
  should obviously be fixed. Report it; do not act on it.
- Confirm the real BeatPilot conversion metric via get_metrics before running any
  report — do not default to "Placed Order."
- Only update email_os_config.email_send_slot_pattern if sample size is genuinely
  sufficient (say what threshold you used). Otherwise leave the existing value and
  say why.
- Log klaviyo_audit_last_run_at in email_os_config when done.

Summarize: facts found (with exact numbers and timeframe), possible explanations
(clearly labeled as interpretation), and recommended tests — kept in three separate
sections per the skill's spec.
```

**Note:** because Klaviyo is attached with no per-tool restriction, watch the first 2–3
runs closely by opening the session transcript afterward, not just the pass/fail
status indicator (a green status only means the session didn't crash — it doesn't mean
it behaved as instructed).

---

## Routine 3: BeatPilot Flow Strategy (on-demand, not scheduled)

**Name:** `BeatPilot Flow Strategy`

**Repository:** `dlyle-lgtm/beatpilot-routine`

**Connectors:** Supabase, Google Drive only.

**Trigger:** Schedule — monthly, e.g. the 1st at 8:00 AM, as a cheap catch-up check in
case a lifecycle data source quietly became available since the last run. You can still
click **Run now** any time you want it to check sooner — e.g. right after you wire up a
new Supabase table or Klaviyo sync.

**Prompt:**

```
Run the email-flow-strategy skill from this repository. Check email_flow_registry
first for every flow in the catalog to avoid duplicating existing live flows
(Lead Conversion, Win-Back) or already-drafted strategies. Never call any Klaviyo
tool — this routine has no Klaviyo connector. Produce briefs only for flows that
don't already have a registry entry, and summarize what's blocking each new flow
from moving past strategy_drafted.
```

---

## The one switch that controls unattended sending

`email_os_config.email_auto_send_enabled` (Supabase, table `email_os_config`) is the
entire safety model now that this runs daily with no human in the loop:

- **`'false'`:** fully automated every day through Klaviyo draft build. Campaigns sit
  ready in Klaviyo as drafts; nothing sends without you manually activating it.
- **`'true'` (current setting, as of your request):** fully automated including the
  live send — genuinely zero-touch. Campaigns are built and sent on schedule with no
  review step.

To flip it: `update email_os_config set value = 'true' where key =
'email_auto_send_enabled';` in Supabase, or just tell me and I'll do it. Flip it back to
`'false'` the same way any time you want to pause unattended sending without touching the
routine itself.

Flow segmentation changes, new flow activation, and pricing/positioning changes are the
only things this system still refuses to touch automatically — those live outside the
campaign pipeline entirely and were never gated by this flag.

## Quick setup checklist

1. Confirm all 7 skill files are committed to `dlyle-lgtm/beatpilot-routine`
2. Go to claude.ai/code/routines → New routine
3. Create Routine 1 with Klaviyo included, run it once manually (**Run now**) before
   trusting the daily schedule, and read the actual session transcript afterward —
   confirm `email_auto_send_enabled` is `'false'` and it actually stopped at Draft
4. Create Routine 2, same manual first-run check
5. Create Routine 3 with a monthly schedule (1st of the month), and use **Run now**
   any time you want it to check sooner
6. When ready for fully unattended sending, flip `email_auto_send_enabled` to `'true'`

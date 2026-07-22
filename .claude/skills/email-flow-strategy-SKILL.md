---
name: email-flow-strategy
description: >
  Designs BeatPilot lifecycle email flow strategies (trial onboarding, feature adoption,
  failed payment, cancellation prevention, etc.) as full build-ready briefs, without
  building or activating anything in Klaviyo. Trigger on "build a flow strategy for
  [flow]", "what flows are we missing", "design the trial onboarding flow", or "run the
  flow strategy skill." Distinguishes strategy work (safe to do now, no live data needed)
  from build-readiness (requires confirming the real Klaviyo-side trigger event/property
  exists — never assumed).
---

# Email Flow Strategy Skill

## 1. Purpose

Write complete, Klaviyo-implementable strategy briefs for BeatPilot's missing lifecycle
flows. This is planning work — it does not require live customer data to produce a good
brief, because a brief describes *what should happen*, not a query against real profiles.
What it must never do is claim a flow is ready to build in Klaviyo without separately
confirming the trigger event/property actually exists there.

## 2. Configuration — resolve at runtime

```sql
select key, value from email_os_config
where key like 'lifecycle_table_%' or key in (
  'lifecycle_data_available',
  'drive_folder_flow_strategy',
  'flow_registry_status_values'
);
```

## 3. Flow catalog (priority order for BeatPilot)

| Priority | Flow | Objective | Entry trigger | Required lifecycle key |
|---|---|---|---|---|
| — | Lead Conversion Sequence | *(already live — do not duplicate)* | New lead | n/a |
| 1 | Trial Onboarding (7-day) | Get a new trial user to a meaningful first action | Trial started | lifecycle_table_trial_status |
| 2 | Trial Activation | Nudge toward first real use within trial | Trial started + no activity in N days | lifecycle_table_generation_activity |
| 3 | Feature Adoption | Surface unused high-value features | Ongoing, usage-gap based | lifecycle_table_feature_usage |
| 4 | Trial Ending | Remind before trial expires, drive upgrade | Trial end date approaching | lifecycle_table_trial_status |
| 5 | Trial Expired (no conversion) | Recover a lapsed trial | Trial ended, no subscription | lifecycle_table_trial_status |
| 6 | New Paid Subscriber Onboarding | Orient a new paying customer | Subscription started | lifecycle_table_billing_events |
| 7 | Failed Payment | Recover a failed charge | Payment failed | lifecycle_table_billing_events |
| 8 | Cancellation Prevention | Intervene before voluntary cancellation | Cancellation flow started / downgrade intent | lifecycle_table_churn_risk |
| — | Win-Back Sequence | *(already live — do not duplicate)* | Inactivity/churn signal | n/a |

Before drafting anything, query `email_flow_registry` for existing rows matching each flow
name (fuzzy match — e.g. "Trial Onboarding" vs "7-Day Trial Onboarding") to avoid
duplicating strategy work already done or a live flow already covering the same job. If
"New Lead Welcome" or "Churn Win-Back" come up as requests, point to the existing Lead
Conversion / Win-Back flows and suggest running the Klaviyo Audit skill on them instead of
designing a replacement from scratch.

## 4. What every brief must contain

For each flow, produce (matching the fields the Audit and Copywriting skills expect):
1. Flow name, objective, entry trigger, trigger filters, profile filters, exit conditions
2. Suppression rules (who should never enter, e.g. already-cancelled, already-converted)
3. Message timing (delay between each email, in real terms — "Day 1," "Day 5," not vague)
4. Branching logic if the flow needs to diverge based on behavior
5. Target customer segment(s)
6. Number of emails and the specific purpose of each one
7. Subject-line direction per email (not final copy — that's the Copywriting skill's job
   once this brief is approved and handed off)
8. CTA per email
9. Personalization variables needed — mark each one as "available now" or "needs
   [lifecycle_table_x]" per the config check in section 2
10. Testing plan (what's worth A/B testing once live)
11. Success metrics
12. Klaviyo implementation instructions (trigger type, filter conditions, flow structure)
    written as if handing this to a Claude Code session with Klaviyo write access

## 5. Build-readiness — separate from strategy completeness

A brief can be fully written and still not be buildable. Before marking anything beyond
`strategy_drafted`:
- Check whether the flow's required lifecycle key (section 3 table) has
  `lifecycle_data_available = 'true'` AND a real (non-empty) value for its specific
  `lifecycle_table_*` config entry.
- If not, the flow strategy is written but the flow registry status stays
  `strategy_drafted`, with a note naming exactly which table/event is still missing.
- Do not additionally assume the data, once it exists in Supabase, is automatically
  present as a Klaviyo profile property or event — that's a separate sync step (Stripe →
  Klaviyo or BeatPilot app → Klaviyo API) that must be confirmed separately before a flow
  actually moves to `building`. State this distinction explicitly in every brief's
  implementation section rather than implying Supabase data readiness = Klaviyo readiness.

## 6. Output

For each flow:
1. Save the full brief as a Google Doc in `drive_folder_flow_strategy`, titled
   `{flow_name} — Strategy Brief`.
2. Upsert `email_flow_registry`: `flow_name`, `objective`, `entry_trigger`, `num_emails`,
   `priority`, `status = 'strategy_drafted'`, `notes` (doc link + missing-data statement +
   Klaviyo-sync caveat from section 5).

Present a summary table to the user: flow name, status, what's blocking it from
`building` (if anything), and a link to the brief.

## 7. Hard rules

- Never call any Klaviyo write endpoint. This skill produces documents and registry rows
  only.
- Never mark a flow `building` or `live` — those transitions happen only when a human (or
  an explicitly-approved Claude Code session with Klaviyo write access) actually
  implements it.
- Never imply that a Supabase lifecycle table existing means the same data is available
  inside Klaviyo as a trigger-ready event/property — these are separate systems and the
  sync between them is a distinct, unconfirmed step every time.
- Never duplicate strategy work for a flow that already exists live — check the registry
  first, every run.
- Do not invent specific numeric targets (e.g. "this will recover 20% of failed
  payments") — success metrics should name *what to measure*, not a fabricated expected
  result.

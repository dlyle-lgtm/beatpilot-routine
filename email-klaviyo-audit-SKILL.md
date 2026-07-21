---
name: email-klaviyo-audit
description: >
  Reads real Klaviyo campaign and flow performance (opens, clicks, conversions,
  unsubscribes, spam complaints, revenue) and produces a prioritized audit — read-only,
  never edits, sends, pauses, or creates anything in Klaviyo. Trigger on "run the Klaviyo
  audit", "how are my emails performing", "audit the last 90 days", or "what's working in
  Klaviyo." Feeds real data back into email_os_config (replacing placeholder assumptions
  like the default send-time slots) and into email_campaign_tracker / email_flow_registry.
---

# Klaviyo Audit Skill

## 1. Purpose

Give an honest, data-grounded read on how BeatPilot's live campaigns and flows are
actually performing, then push real numbers back into the config other skills rely on —
so the Calendar skill's send-time assumption and the Ideation/Copywriting skills'
"what's working" context stop being guesses.

**This skill only reads.** It never creates, edits, sends, pauses, or cancels a campaign
or flow. Klaviyo's campaign/flow read tools render an approval prompt on first use in a
session — if declined, stop and tell the user rather than retrying or working around it.

## 2. Configuration — resolve at runtime

```sql
select key, value from email_os_config
where key in (
  'klaviyo_conversion_metric_name',
  'klaviyo_audit_lookback_default',
  'email_send_slot_pattern'
);
```

## 3. Step 0 — confirm the conversion metric before anything else

BeatPilot is a SaaS subscription product, not e-commerce — the tools' own default fallback
("Placed Order") is an e-commerce assumption and is almost certainly wrong here. Before
running any report:

1. Call `get_metrics` (no extra filters) and look for the real metric that represents a
   meaningful conversion for BeatPilot — e.g. a trial-start, subscription, or upgrade
   event.
2. If no such metric exists yet, tell the user which metric names ARE available and ask
   which one should count as "conversion" for this audit — do not silently default to
   "Placed Order" for a product that doesn't sell physical goods.
3. Once confirmed, update `email_os_config.klaviyo_conversion_metric_name` so future runs
   don't re-ask.

## 4. Campaign audit

```
get_campaign_report(
  conversion_metric_id = <resolved above>,
  statistics = [open_rate, opens_unique, click_rate, click_to_open_rate,
                conversion_rate, conversion_uniques, unsubscribe_rate,
                spam_complaint_rate, bounce_rate, delivery_rate],
  timeframe = { key: klaviyo_audit_lookback_default },
  group_by = [campaign_id, campaign_message_id],
  group_by_audience = true
)
```

Cross-reference returned `campaign_id`/`campaign_message_id` against
`email_campaign_tracker.klaviyo_campaign_id` to match performance back to the idea and
copy that produced it. Where a match exists, write `results` (jsonb) on that tracker row.

## 5. Flow audit

```
get_flow_report(
  conversion_metric_id = <resolved above>,
  statistics = [open_rate, click_rate, click_to_open_rate, conversion_rate,
                conversion_uniques, unsubscribe_rate, spam_complaint_rate],
  timeframe = { key: klaviyo_audit_lookback_default },
  group_by = [flow_id, flow_message_id]
)
```

Match `flow_id` against `email_flow_registry.klaviyo_flow_id` (backfill that column if
currently null, using `get_flows` to resolve names → IDs) and update each flow's `notes`
and `status` — flag any flow that exists in Klaviyo but not in the registry, and vice
versa, as a data-integrity note rather than silently reconciling one direction only.

## 6. Segment context

Call `get_segments` to resolve segment IDs/names referenced in the reports above, so
audit output speaks in segment names, not opaque IDs.

## 7. Deriving real send-time data (fixes the Phase 4 placeholder)

From the campaign report, group performance by day-of-week/time-of-send (available via
each campaign's `send_time`/`scheduled_at` metadata from `get_campaigns` cross-referenced
with report stats). If there's enough sample size (the skill should say what "enough"
means for this account, e.g. at least 4 sends per slot) to identify a real best-performing
send window, update `email_os_config.email_send_slot_pattern` with the real value and
change its `notes` field to state it's now data-derived, with the date of derivation and
sample size. **If sample size is too small to draw a conclusion, say so explicitly and
leave the placeholder in place** — do not present a low-confidence guess as settled data.

## 8. Output structure — keep these three categories separate

1. **Facts found in the data** — exact numbers, plainly stated, with the timeframe and
   sample size.
2. **Possible explanations** — clearly labeled as interpretation, not fact.
3. **Recommended tests** — specific, actionable, tied to `email_experiment_log` schema
   (hypothesis, variable_tested, etc.) so a recommendation can become a real experiment row
   once the user approves running it.

Also produce:
- Executive summary (3–5 sentences)
- Critical issues (e.g. any campaign/flow with unsubscribe_rate or spam_complaint_rate
  meaningfully above account average — state the actual comparison, don't invent a
  universal "good" benchmark)
- Low-hanging opportunities (segments or flows with a clear, fixable gap)
- Prioritized recommendations with expected impact and effort, in plain terms

## 9. Hard rules

- Never call any Klaviyo write endpoint (create/update/send/cancel/pause) — this skill has
  no legitimate reason to touch one, and must not attempt it even if asked to "just fix"
  something it finds. Report the issue; let the user or a separate approved step act on it.
- Never invent a benchmark ("industry average open rate is X%") unless the user supplies
  one — compare BeatPilot's own campaigns/flows against each other and against their own
  historical trend, not against unsourced external numbers.
- Never overwrite `email_os_config.email_send_slot_pattern` on thin sample size — say so
  and leave the existing value/notes alone.
- If the Klaviyo report tool prompts for approval and the user declines, stop, tell them
  plainly, and do not retry or attempt an alternate path to the same data.

## 10. Output

Log `klaviyo_audit_last_run_at` in `email_os_config` on completion. Present the audit to
the user as the four sections in step 8 — do not reorganize into a different report
format per run, so audits stay comparable month over month.

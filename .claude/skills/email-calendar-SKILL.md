---
name: email-calendar
description: >
  Takes approved email copy from Supabase `email_campaign_tracker` (copy_status='approved')
  and slots it into an actual send calendar — assigning dates/times, enforcing frequency
  caps and fatigue rules, avoiding blackout dates, and logging every scheduled item to the
  Beat Pilot Google Calendar per standing rule. Trigger on "build the send calendar",
  "schedule approved emails", "run the calendar skill", or "what's the email calendar look
  like for [month]". Does NOT create anything in Klaviyo — scheduling here is planning
  only; the actual Klaviyo campaign build is a separate, explicitly-approved step.
---

# Email Calendar Skill

## 1. Purpose

Turn a backlog of approved-but-unscheduled copy into a concrete calendar: which campaign
goes out which day/time, respecting frequency limits, subscriber fatigue, and blackout
dates. Writes the schedule to `email_campaign_tracker` and creates a matching event on the
Beat Pilot Calendar for every item — never silently schedules something the user can't see
on their calendar.

## 2. Configuration — resolve at runtime

```sql
select key, value from email_os_config
where key in (
  'email_send_slot_pattern',
  'email_frequency_cap_per_week',
  'email_min_gap_days_same_segment',
  'email_min_gap_days_same_funnel_stage',
  'email_max_consecutive_promotional',
  'email_blackout_dates',
  'calendar_id_beat_pilot'
);
```

**Say this out loud at the start of every run:** `email_send_slot_pattern` is a
best-practice default, not derived from BeatPilot's own open/click data — flag it as an
assumption until the Klaviyo Audit skill (Phase 5) supplies real send-time performance and
this config value gets updated with actual data. Never present the default slots as if
they were data-backed.

## 3. Input selection

```sql
select * from email_campaign_tracker
where copy_status = 'approved'
and scheduled_date is null
order by created_at asc;
```

## 4. Scheduling logic

1. Walk the queue in order, assigning the next available slot from
   `email_send_slot_pattern`.
2. Enforce `email_frequency_cap_per_week` — never assign more sends in a rolling 7-day
   window than the cap allows.
3. Enforce `email_min_gap_days_same_segment` — check already-scheduled rows in
   `email_campaign_tracker` (and rows being scheduled in this same run) for the same
   `target_segment`; push to a later slot if the gap is too short.
4. Enforce `email_min_gap_days_same_funnel_stage` — same check against `funnel_stage`.
5. Enforce `email_max_consecutive_promotional` — if the last N scheduled sends were all
   `email_type = 'promotional'` (or urgency/last-chance types), the next promotional send
   gets pushed and a market-intel/educational item is pulled forward instead, if one is
   available in the queue.
6. Skip any date present in `email_blackout_dates`. If skipping causes a cap violation
   (e.g. every slot that week is blacked out), tell the user rather than silently
   overriding the blackout.
7. If two items are equally eligible for the same slot, prioritize by
   `email_campaign_tracker` → back-reference `email_ideas.priority_score` (join on
   `idea_id`) — higher priority goes first.

## 5. Conflict flagging

Before finalizing, check whether any scheduled date falls within 3 days of a known product
launch or major promotion if the user has told you about one in this conversation or a
prior one. If you don't have that context, say explicitly that launch-conflict checking is
based only on what's been mentioned to you, not a live launches table (none exists yet) —
don't imply broader awareness than you have.

## 6. Output

For each newly scheduled row, update `email_campaign_tracker`:
- `scheduled_date` (assigned slot, in UTC)
- `next_action = 'ready for Klaviyo build — requires explicit approval before sending'`

Then create one Google Calendar event per scheduled campaign on
`calendar_id_beat_pilot`, per the standing full-context rule:
- Title: `[Email] {campaign_name} — {email_type}`
- Description includes: target_segment, funnel_stage, copy doc link (from
  `copy_doc_url`), current status (scheduled, not yet built in Klaviyo), and a one-line
  reminder that this is a plan, not a live send
- Store the returned event id back into `email_campaign_tracker.calendar_event_id`

Present the resulting calendar to the user as a simple date-ordered table (date/time,
campaign, type, segment) before finishing, and call out anything that got pushed due to a
frequency/gap/blackout rule so nothing silently slips a week without the user noticing.

## 7. Hard rules

- Never create, update, or send anything in Klaviyo. This skill only plans dates and logs
  calendar events.
- Never present the default send-time slots as account-derived — always label them as
  best-practice defaults until Phase 5 (Audit skill) supplies real data.
- Never override a blackout date, even to hit a frequency target — tell the user instead.
- Never schedule two same-segment or same-funnel-stage sends closer than the configured
  minimum gap, even under pressure to "fit everything into 30 days."
- If Google Calendar event creation fails for a given item, still keep the
  `scheduled_date` update in `email_campaign_tracker`, tell the user the calendar entry is
  missing, and don't fail the rest of the batch.

## 8. Graceful degradation

If `email_campaign_tracker` has no `approved` + unscheduled rows, say so and suggest
running the Copywriting skill first rather than fabricating a schedule. If
`email_blackout_dates` or the slot pattern config is missing, use safe defaults (no
blackout dates, twice-weekly cadence) and say explicitly that you're doing so.

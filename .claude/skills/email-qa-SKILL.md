---
name: email-qa
description: >
  Runs a final quality-assurance checklist on scheduled, approved email copy before it's
  considered ready to hand off for Klaviyo build — checks audience/suppression logic,
  personalization, claims accuracy, offer/date correctness, and flags anything a human
  needs to verify visually (rendering, live links) that this skill cannot check itself.
  Trigger on "QA this campaign," "run the QA skill," or automatically as the last step
  before the orchestrator marks something ready for build. Never edits copy itself —
  flags issues back to the Copywriting or Calendar skill.
---

# Email QA Skill

## 1. Purpose

Be the last checkpoint before a campaign is called "ready for Klaviyo build." Catch
structural and factual problems this skill *can* verify from the data it has, and clearly
name the things it *can't* verify (visual rendering, live link status) so those don't get
silently assumed to be fine.

## 2. Input selection

```sql
select * from email_campaign_tracker
where copy_status = 'approved'
and scheduled_date is not null
and qa_status = 'not_started';
```

## 3. Checklist — things this skill CAN verify from data on hand

Read the copy doc (`copy_doc_url`) and the tracker row, then check:

1. **Audience/segment correctness** — `target_segment` is a real, named segment/list, not
   a placeholder or typo
2. **Send date/time** — `scheduled_date` doesn't fall on a blackout date
   (`email_os_config.email_blackout_dates`) and respects the frequency/gap rules the
   Calendar skill already applied — flag if something looks off despite passing that skill
3. **Sender identity** — matches the confirmed BeatPilot sender ("Beat Pilot")
4. **Subject line + preview text present**, not empty, not literally a placeholder like
   "[SUBJECT HERE]"
5. **No placeholder text** anywhere in the body (scan for brackets, "TODO," "Lorem ipsum,"
   "[insert...]")
6. **Personalization variables** — every merge tag referenced actually maps to a real,
   available field (cross-check against the Copywriting skill's "available now" vs "needs
   lifecycle data" flags); flag any merge tag that assumes data that doesn't exist yet
7. **Offer/CTA correctness** — trial vs. paid-tier vs. thumbnail-credit-pack language
   matches the actual `funnel_stage` and doesn't blur the three. **Specific hard check:**
   if `target_segment` is "Full subscriber list (no lifecycle segmentation available
   yet)" or `funnel_stage = 'all_active'`, the CTA must be segment-neutral (product/
   dashboard-pointed) — fail QA immediately if it says "Start Your Free Trial" or any
   other language that assumes the recipient hasn't already converted, since that
   segment includes paying customers.
8. **Claims accuracy** — any statistic or claim in the copy is traceable to the idea's
   `rationale` (from `email_ideas`) or an explicit brand doc source; flag anything that
   reads like an invented number
9. **Footer/unsubscribe language present** (standard, not a new compliance claim)
10. **Grammar/spelling pass** on the copy doc

## 4. Checklist — things this skill CANNOT verify, must flag for human review

State these plainly every time rather than silently skipping them:
- Mobile/desktop/dark-mode rendering — this skill has no rendering preview; recommend
  using Klaviyo's own preview/test-send feature before the human approves
- Live link functionality — this skill does not click links; recommend a manual click-
  through or Klaviyo's link-check on test send
- Actual deliverability at send time (spam folder placement, etc.)

## 5. Output

For each campaign checked, update `email_campaign_tracker`:
- `qa_status = 'passed'` only if every item in section 3 is clean
- `qa_status = 'failed'` with `qa_notes` listing every specific issue found (not just
  "some issues") if anything is wrong
- `qa_checked_at = now()`

Present results grouped by pass/fail. For failures, name the exact issue and which skill
should fix it (copy issue → Copywriting skill; scheduling issue → Calendar skill). Always
restate the section 4 items that need human eyes, even for a `passed` result — a QA pass
here does not mean "safe to send without looking at it."

## 6. Hard rules

- Never edit the copy or the schedule directly — flag issues back, don't silently fix them.
- Never mark `qa_status = 'passed'` if any section 3 item failed.
- Never let a `passed` result imply visual/link verification happened — always restate
  that those remain manual steps.
- Never call Klaviyo. This skill only reads the tracker and the copy doc.

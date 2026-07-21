---
name: email-copywriting
description: >
  Turns approved ideas from Supabase `email_ideas` into finished BeatPilot email copy —
  subject lines, preview text, full body, plain-text version, CTA, and testing notes —
  saved as a Google Doc and logged to `email_campaign_tracker`. Trigger on "write the
  copy for idea #[n]", "run the copywriting skill", "turn approved ideas into emails", or
  any request to draft email copy from an already-approved idea. Never sends or builds
  anything in Klaviyo — output is a drafted doc awaiting human approval.
---

# Email Copywriting Skill

## 1. Purpose

Take one or more rows from `email_ideas` where `status = 'approved'`, write full campaign
copy in BeatPilot's voice, save it as a Google Doc, and log everything to
`email_campaign_tracker` with `copy_status = 'drafted'`. This skill drafts only — it never
approves its own copy and never touches Klaviyo.

## 2. Configuration — resolve at runtime, never hardcode

```sql
select key, value from email_os_config
where key in (
  'brand_guide_authoritative_doc_id',
  'voice_guide_doc_id',
  'brand_bible_doc_id',
  'drive_folder_campaign_copy'
);
```

Read `voice_guide_doc_id` and `brand_bible_doc_id` via Drive `read_file_content` at the
start of every run (not cached from a prior session — brand docs can change) and pull:
- Approved hook system (primary/monetization/pain-first/command/curiosity hooks)
- Feature-language → outcome-language mapping table
- Vocabulary to use / avoid
- Tone-by-channel table (email = "warm mentor")
- Copy patterns (pain→fix, etc.)

## 3. Input selection

```sql
select * from email_ideas
where status = 'approved'
and id not in (select idea_id from email_campaign_tracker where idea_id is not null)
order by priority_score desc;
```

If the user named specific idea numbers from a prior Ideation batch, filter to those.
Otherwise process all unclaimed approved ideas.

## 4. Format map (email_type → structure)

| email_type | Structure |
|---|---|
| plain_text_founder | No design, short, from "Beat Pilot" sender identity, one clear ask, feels personal |
| educational | Headline, teaching hook, worked example (real data point from the idea), single CTA |
| promotional | Offer-led, urgency only if genuinely true (real trial/pricing deadline), clear CTA |
| myth_busting / comparison | Pain-first hook, "us vs. the wrong way" framing, proof, CTA |
| product_announcement | What changed, why it matters to the producer, CTA to try it |
| customer_story | Real testimonial only if one exists in reference material — never invented |
| urgency / re_engagement | Only used with a real time-bound reason; never manufactured scarcity |

## 5. What to generate per idea

For each approved idea, produce:
1. 3–5 candidate subject lines (pull from the approved hook system where the angle fits;
   don't force a hook that doesn't match the idea's actual content)
2. Preview text
3. Headline / opening hook
4. Full body copy (outcome language, not feature language — translate through the mapping
   table before writing)
5. Primary CTA text, matched to funnel_stage:
   - new_lead / trial → "Start Your Free Trial"
   - active trial → in-product action, not "buy" language
   - active paid → feature/upsell CTA, never re-pitch the trial
   - at_risk / churned → categories currently disabled per the Ideation skill; do not draft
     these until lifecycle mode is live (see email-ideation SKILL.md section 6)
6. Optional secondary CTA (only if it doesn't compete with the primary one)
7. Plain-text version (no HTML/design assumptions)
8. Personalization recommendations (fields to merge-tag, e.g. first name, niche/genre from
   their account if available — flag as "requires lifecycle data" if it's not available yet)
9. Compliance footer note (standard unsubscribe/footer, no new compliance claims invented)
10. 1–2 testing recommendations (e.g. which subject line to A/B, log as a candidate for
    `email_experiment_log` — don't create the experiment row yet, just note the recommendation)

## 6. Hard rules

- Match the approved brand voice exactly — direct, confident, charged, empathetic
  underneath, producer-native. No generic SaaS phrasing ("leverage," "unlock," "seamless").
- Never fabricate a statistic, testimonial, or product capability. If the idea's rationale
  cites a real number from `niche_opportunities`/`artist_intelligence`, that number can be
  used verbatim; anything else must come from the user or existing brand docs.
- Never use false urgency — only reference deadlines that are real (actual pricing changes,
  actual trial expiration windows once lifecycle data exists).
- One primary objective per email. The CTA must match that objective.
- Clearly distinguish free trial vs. paid plan (Starter/Growth/Pro) vs. one-time thumbnail
  credit packs — never blur these into a single generic "buy now."
- Do not repeat the same hook or subject-line angle used in the last 30 days — check
  `email_campaign_tracker.subject_lines` and `email_ideas.main_angle` for recent overlap
  before finalizing.
- Never call Klaviyo. Never mark copy as sent or scheduled. This skill only drafts.

## 7. Output

1. Create a Google Doc in the folder at `email_os_config.drive_folder_campaign_copy`,
   titled `{YYYY-MM-DD} — {campaign_name}`, containing all fields from section 5 in a
   clearly labeled, skimmable layout (not raw prose).
2. Upsert into `email_campaign_tracker`:
   - `idea_id`, `campaign_name`, `email_type`, `target_segment`, `funnel_stage`
   - `subject_lines` (jsonb array of the candidates)
   - `preview_text`
   - `copy_status = 'drafted'`
   - `copy_doc_url` = the new Doc's URL
   - `next_action = 'awaiting human copy approval'`
3. Present a short summary to the user per campaign (campaign name, subject line options,
   doc link) and ask which to approve. On approval, set `copy_status = 'approved'` — that's
   the signal the Calendar skill (Phase 4) picks up.

## 8. Graceful degradation

If the Drive read for either brand doc fails, stop and tell the user which doc couldn't be
read rather than drafting copy from memory of a prior version — brand voice can change
between sessions and stale copy is worse than a short delay. If `email_ideas` has no
`approved` rows, say so and suggest running the Ideation skill first instead of inventing
an idea on the spot.

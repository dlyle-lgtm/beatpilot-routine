---
name: email-ideation
description: >
  Generates BeatPilot email campaign ideas across seven categories (market intelligence,
  product education, feature adoption, trial conversion, customer retention, win-back,
  product announcements) and writes them to Supabase `email_ideas` for approval. Trigger
  on "generate email ideas", "run the ideation skill", "give me campaign ideas for
  [month/week]", or any request for email/newsletter content ideas. Market-intelligence,
  product-education, and product-announcement categories are fully operational today.
  Feature-adoption, trial-conversion, retention, and win-back categories are prepared but
  stay disabled until the required customer lifecycle tables exist (see section 6) — the
  skill checks for them at runtime and never fabricates lifecycle behavior it can't see.
---

# Email Ideation Skill

## 1. Purpose

Turn live BeatPilot data into a numbered list of email campaign ideas the user can approve
by number. This skill never writes copy and never touches Klaviyo — it only proposes ideas
and logs them to `email_ideas` (status `pending`) for the Copywriting skill to pick up once
approved.

## 2. Configuration — read these before every run, never hardcode table names

Query `email_os_config` at the start of every run:

```sql
select key, value from email_os_config
where key in (
  'source_table_niche_opportunities',
  'source_table_artist_intelligence',
  'lifecycle_data_available',
  'lifecycle_table_trial_status',
  'lifecycle_table_onboarding_progress',
  'lifecycle_table_feature_usage',
  'lifecycle_table_generation_activity',
  'lifecycle_table_billing_events',
  'lifecycle_table_login_activity',
  'lifecycle_table_email_engagement',
  'lifecycle_table_churn_risk'
);
```

Use the returned `value` as the actual table/view name in every query below — never write
`niche_opportunities` or `artist_intelligence` literally in generated SQL. This lets the
user rename, view-ify, or repoint any source without editing this skill.

If a config row is missing entirely, treat that source as unavailable (same as an empty
string value) and skip anything that depends on it.

## 3. Campaign category status map

| # | Category | Status | Source(s) |
|---|----------|--------|-----------|
| 1 | Market intelligence | **Enabled** | niche_opportunities, artist_intelligence |
| 2 | Product education | **Enabled** | niche_opportunities, artist_intelligence, product knowledge (static) |
| 3 | Product announcements | **Enabled** | user-supplied announcement input, no table dependency |
| 4 | Feature adoption | Disabled — needs feature_usage | lifecycle_table_feature_usage |
| 5 | Trial conversion | Disabled — needs trial_status | lifecycle_table_trial_status |
| 6 | Customer retention | Disabled — needs churn_risk + feature_usage | lifecycle_table_churn_risk, lifecycle_table_feature_usage |
| 7 | Win-back | Disabled — needs billing_events + login_activity | lifecycle_table_billing_events, lifecycle_table_login_activity |

At the start of every run, print this table with live status (checked against
`email_os_config`) so the user always sees what's actually active before ideas generate.
**Never silently skip a category — always name it and say why.**

## 4. Market Mode (fully operational)

### 4a. Market Intelligence category

Pull the top opportunities and momentum signals:

```sql
-- Opportunity signals
select niche_phrase, parent_genre, opportunity_score, competition_level,
       growth_potential, reason, mention_count, average_views,
       competing_video_count, opportunity_score_change_7d, score_delta
from {source_table_niche_opportunities}
where opportunity_score is not null
order by opportunity_score desc, opportunity_score_change_7d desc nulls last
limit 20;

-- Artist/momentum signals
select artist_name, genre_bucket, trend_score, trend_status,
       mention_count, average_views, highest_views
from {source_table_artist_intelligence}
where trend_status in ('rising', 'breaking', 'hot')
order by trend_score desc
limit 20;
```

For each strong signal (high `opportunity_score`, rising `score_delta`, low
`competition_level`, or `trend_status` = rising/breaking), draft an idea that:
- Names the specific niche/genre/artist and the real numbers behind it
- Frames it as "here's what's moving right now and what to do about it"
- Ties the insight directly to a BeatPilot feature that surfaced it (Opportunity Engine,
  Shorts Signals, trend analysis — pull exact feature names/outcome language from the
  authoritative Brand Guide voice section, doc id in `email_os_config.brand_guide_authoritative_doc_id`)
- Never states a number that isn't directly in the query result

### 4b. Product Education category

Same source tables, different framing: instead of "here's a hot opportunity," explain a
concept or workflow (e.g. "what opportunity_score actually measures and how to use it,"
"reading competition_level before you commit to a beat") using real current data as the
worked example. Draw copy patterns and outcome-language mappings from the Brand Voice Guide
(`email_os_config.voice_guide_doc_id`) — outcomes, never features, as the lead.

### 4c. Product Announcements category

No table dependency. Takes a user-supplied announcement (new feature, pricing change, UI
update) as direct input and drafts the announcement angle, audience, and CTA. Ask the user
for the announcement details if none were provided in the trigger message — this is the one
category that needs a quick human input rather than pulling from data.

## 5. Idea generation rules (all categories)

1. Build the next 30 days of ideas per run unless the user specifies a different window.
2. Default cadence: 2 campaigns/week unless told otherwise.
3. Do not repeat the same primary topic (same niche_phrase, artist_name, or feature) within
   30 days — check existing `email_ideas` and `email_campaign_tracker` rows first.
4. Every idea must cite the specific data point it's based on (score, trend_status, etc.) in
   the `rationale` field — no idea should read as generic.
5. Never invent a statistic, testimonial, or feature capability not present in source data
   or explicitly told to you by the user.
6. Flag `similarity_flag` if an idea closely resembles one built in the last 30 days.
7. Assign `priority_score` (1–10) weighted toward: high opportunity_score, low
   competition_level, rising score_delta/trend_status, and category currently enabled.

## 6. Lifecycle Mode (prepared, inactive until data exists)

**Do not write or run any lifecycle query until `email_os_config.lifecycle_data_available`
= 'true'.** Until then, when the user asks for feature adoption, trial conversion,
retention, or win-back ideas, respond that the category is disabled and name the exact
missing table(s) from section 3 — do not attempt to approximate the answer from market
data, and do not guess at customer behavior.

### Required lifecycle data (not yet built — for future reference)

| Data needed | Powers category | Suggested source |
|---|---|---|
| Trial stage, subscription status, trial expiration date | Trial conversion, win-back | New `trial_status` table or Stripe sync |
| Onboarding progress | Trial conversion | New `onboarding_progress` table |
| Last login | Retention, win-back | New `login_activity` table |
| Feature usage, unused features | Feature adoption, retention | New `feature_usage` table |
| Generation activity (beats/thumbnails) | Feature adoption | New `generation_activity` table |
| Cancellation, failed payment | Win-back | New `billing_events` table (Stripe webhook sync likely) |
| Email engagement (opens/clicks/last engaged) | Retention, win-back | Klaviyo API export, not necessarily Supabase |
| Churn risk score | Retention | Computed/derived table once the above exist |

Once any of these tables exist, add its real name to `email_os_config` under the matching
`lifecycle_table_*` key and flip `lifecycle_data_available` to `'true'`. This skill will
pick it up automatically on the next run — no code change needed here, only config rows.
Enable categories incrementally: a category goes live only when *all* of its required
tables in section 3 are present, not just some of them.

## 7. Output

Write each approved-for-review idea as a row in `email_ideas` (status `pending`) with all
fields populated: `campaign_name`, `email_type`, `main_angle`, `customer_problem`,
`core_promise`, `product_feature`, `target_segment`, `funnel_stage`, `suggested_format`,
`main_cta`, `rationale`, `priority_score`, `similarity_flag`.

Then present the batch to the user as a numbered list (20–30 ideas per the standard "generate
ideas" command) grouped by category, with priority_score visible, so they can reply with the
numbers they approve. On approval, update `status` to `approved` for those rows — everything
else stays `pending` or gets marked `rejected` if the user says so.

## 8. Hard rules

- Never call Klaviyo directly. This skill proposes ideas only.
- Never fabricate lifecycle customer behavior — if the data isn't in a configured table,
  say so and stop for that category.
- Never hardcode a table name — always resolve through `email_os_config` at runtime.
- If a configured source table is unreachable or empty, skip only that source, note it in
  the status table output, and continue using whatever valid sources remain. Do not fail
  the whole run.
- Log which config values were used at the top of every run's output for traceability.

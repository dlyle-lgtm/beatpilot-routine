---
name: social-campaign-planner
description: Plan BeatPilot's social media campaigns and content calendar — reviews the content backlog, brand state, and recent performance, then either hands off routine posting instructions to `social-poster` directly or creates an approval card in the Beat Pilot Notion board for anything net-new or risky (new campaign concepts, new platforms, anything referencing pricing or customer results). Use when asked to plan social content, plan a campaign, build a content calendar, or run the social media planner.
---

# Social Campaign Planner (BeatPilot)

Plan what gets posted and when. This skill does not post anything itself — `social-poster` handles execution. Its job is to turn the content backlog and brand/business context into a concrete plan, and to explicitly separate what's safe to auto-post from what needs a human look first.

## Read before planning

- **Google Drive**: Topics - Final spreadsheet, Producer Tips spreadsheet + Used-Topics-Log (what's queued, what's already gone out), BeatPilot-Info folder (`1KF05eVO3FVotXQ5oD_KgnSRMTrjwEQ41`) for brand guidelines and real product grounding — specifically `BeatPilot_Producer_Benefits_Guide.docx` (feature-to-benefit mapping, confirmed pricing: Starter $12/mo, Growth $24/mo, Pro $49/mo), `BeatPilot — Full Feature Audit.docx` (the actual current feature inventory — Opportunity Engine, Confidence Score, Market Imbalance, Upload Timing Heatmap, Type Beat Leaderboard, Thumbnail Studio/Analyzer, My Studio, Shorts Signals), `BeatPilot_Target_Demographic_Customer_Profile.docx` (real audience specifics), and `BeatPilot_Paid_Advertising_Strategy.docx` (existing ad positioning). Never invent a feature name or plan around a guessed one — the Full Feature Audit is the only source of truth for what BeatPilot actually does.
- **The Social Media Log** (a doc in BeatPilot-Info, parallel to the CEO Journal — create it if it doesn't exist) — what's been posted and planned recently, so this run doesn't repeat itself or contradict a prior plan.
- **Blotato**: recent post performance on the connected account(s) — right now that's Instagram only.
- **Higgsfield**: credit balance and usage, as available generation capacity for the plan.
- **The Beat Pilot Notion board** — check open `CEO Task` cards relevant to marketing (distribution expansion, brand collisions, idle-credit usage, testimonial audit) so the plan stays aligned with what the CEO system has already flagged rather than duplicating or contradicting it.

## Standing guardrails — apply these to every plan, no exceptions

- **Platforms**: Instagram only, until the `@beatpilotnow` brand-collision question is resolved (two open Notion cards currently contradict each other on whether that handle is BeatPilot's own). Never plan TikTok or YouTube Shorts content until that's explicitly resolved and confirmed.
- **Claims**: BeatPilot has zero paying customers as of the last check. Never plan content that references customer testimonials, results, "join thousands of producers," or similar social proof that isn't independently verified as real.
- **Pricing**: only reference specific dollar amounts confirmed in `BeatPilot_Producer_Benefits_Guide.docx` (Starter $12/mo, Growth $24/mo, Pro $49/mo) — not from memory, assumption, or an invented figure.

## Decide what's routine vs. what needs approval

**Routine — hand off to `social-poster` directly, no Notion card needed:**
- Continuing to draw from the Producer Tips or Topics - Final backlog in an established format
- Reusing an already-proven carousel template or post structure
- Normal scheduling within the existing cadence (near-daily, six ET slots, Wednesday/Thursday weighted heavier)

**Needs approval — create a Notion card, do not let `social-poster` act until it's moved to "In progress":**
- A new campaign concept or content format that hasn't been used before
- First use of Higgsfield-generated video creative (or any new creative direction)
- Anything that touches pricing, claims, or testimonials
- Any expansion to a new platform
- Anything the planner itself is unsure is safe to auto-post

## Output

For the routine portion: write a short plan directly into the Social Media Log (what's queued for the period, which backlog items it draws from, target dates/slots) — `social-poster` reads this log to know what to execute.

For anything needing approval, create a Notion card with `notion-create-pages` under `parent: {"data_source_id": "collection://3932ee00-89fb-800a-a8e6-000bf3b9f20d"}`:

- `"Project name"`: specific, e.g. "Approve: first Higgsfield video-ad creative for paid Meta campaign"
- `"Status"`: `"Not started"`
- `"Priority"`: judged on timing/impact
- `"Team"`: `"Business Development"` (best fit for marketing content)
- `"Social Task"`: `"__YES__"` — always set this on cards this skill creates. Never set `"CEO Task"`.

Page content structure:

## What this is
The specific content/campaign idea, described concretely.

## Why it's flagged for approval
Which guardrail or novelty triggered the approval requirement.

## What's ready
Exactly what's been prepared (draft copy, generated creative, proposed schedule) so approving the card is a real "go" decision, not a vague concept.

## Once approved
What social-poster will do the moment this moves to "In progress."

## Guardrails

- Never post anything directly — that's `social-poster`'s job, and only after approval where required.
- Never plan around unverified claims, retired branding, or unresolved platform collisions, even if asked.
- Keep the Social Media Log entry short and dated — a running plan, not a report.
- If the backlog runs low or brand guidelines seem stale, say so plainly rather than planning around a gap silently.

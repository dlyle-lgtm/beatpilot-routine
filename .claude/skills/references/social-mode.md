# Social Campaign Mode

Use this mode only after a post is confirmed live (Publishing Preparation Mode complete). This mode prepares a campaign — it must never schedule or post without an explicit human approval, per Mandatory Rule 7.

## 1. Platform-specific captions

Write a distinct caption per platform rather than reusing one caption everywhere — match each platform's norms:
- Instagram / Facebook: hook line first, short paragraph, one clear CTA, relevant hashtags at the end.
- X / Threads: shorter, punchier, can lead with the data point or the pain-first hook directly.
- TikTok / YouTube Shorts / Reels: caption supports the video/visual, not a replacement for it — keep it brief.
- LinkedIn: slightly more professional framing, can lean into the "producer as business owner" angle from the Brand Guide's positioning.
- Pinterest: description-style, keyword-rich, since Pinterest functions more like search than social.

Every caption should use the assigned hook/CTA from `BeatPilot CTA Library` matching the article's topic — don't write a new one from scratch each time.

## 2. UTM links

Build a tracked link for every platform/post combination, e.g.:
`https://www.beatpilot.io/blog/[slug]?utm_source=[platform]&utm_medium=social&utm_campaign=[topic-id-or-slug]`

Keep the `utm_campaign` value consistent across all platforms promoting the same article so performance can be compared apples-to-apples later (this feeds Content Refresh Mode's ranking/performance review).

## 3. Social calendar / posting slots

Check `06 Social Campaigns / Scheduled Campaigns` and whatever social log already exists (e.g. a Social Media Log or Producer Tips-style used-topics log) for what's already queued, and find the next open slot rather than double-booking a time. Respect whatever posting cadence/schedule is already established for the account instead of inventing a new one.

## 4. Prepare the Blotato campaign

Draft the campaign (captions, images/video, UTM links, proposed schedule) and save it to `06 Social Campaigns / Draft Campaigns`. Do not call any Blotato scheduling/posting tool yet — that only happens after step 5.

## 5. Human approval gate

Present the full draft campaign to the user — every caption, every platform, every scheduled time — and get explicit approval before doing anything else. This can be a simple "looks good, go ahead" from the user in the conversation, or a Notion approval-card pattern if that workflow is already in place for this account (a "Social Task" checkbox gate, matching the existing `social-poster`/`social-campaign-planner` skills' pattern, if those are present in this environment).

Once approved:
- Move the campaign from `Draft Campaigns` to `Approved Campaigns`.
- Use Blotato's scheduling tools to actually queue the approved posts.
- Move the campaign to `Scheduled Campaigns`, then `Published Campaigns` once each post goes live.
- Log the run in `10 System Logs / Publishing Logs` (or a dedicated social log if the account has one).

**Never call a Blotato posting/scheduling tool before this approval step has happened in the current conversation.** A prior run's approval does not carry over to a new campaign.

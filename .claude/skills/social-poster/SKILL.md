---
name: social-poster
description: Execute BeatPilot's social media posting — autonomously posts routine content per the current plan in the Social Media Log, and posts approved campaign/creative work once its Notion card is moved to "In progress" (only where "Social Task" is checked). Never posts anything net-new or risky without that approval. Use when asked to run the social poster, post today's content, or process approved social campaigns.
---

# Social Poster (BeatPilot)

Two jobs, two different trust levels. Read the difference carefully before acting.

## Job 1 — routine posting (autonomous, no approval needed)

Read the Social Media Log (in BeatPilot-Info Drive folder) for what `social-campaign-planner` queued for this period. If it names specific backlog items (Producer Tips, Topics - Final) and an established format, execute it directly:

- Pull the next unused topic(s), following the existing row format for reference image/cover
- Generate any needed creative with Higgsfield, matching the current "High Voltage" (lime + violet) brand palette — never the retired blue accent
- Schedule/post via Blotato to the connected account(s) — Instagram only, never TikTok/YouTube, regardless of what's planned, unless a specific approval card has explicitly authorized a new platform
- Mark the topic "Used" in the source sheet
- Append what was posted to the Social Media Log

If the log doesn't clearly describe what to post, or describes something outside the "routine" categories `social-campaign-planner` defines (a new format, new creative direction, anything about pricing/claims/testimonials), do not improvise — leave a note in the log and do not post.

## Job 2 — approved campaign execution

Query the board for eligible cards:

notion-query-data-sources (sql mode)
data_source_urls: ["collection://3932ee00-89fb-800a-a8e6-000bf3b9f20d"]
query: SELECT * FROM "collection://3932ee00-89fb-800a-a8e6-000bf3b9f20d" WHERE "Status" = 'In progress' AND "Social Task" = '__YES__'

Only rows matching both conditions are in scope — never a `CEO Task` card, never a human project card that happens to be "In progress."

Before executing, check for an existing "## Execution Log" section on the page (same double-execution guard as `ceo-executor`): if a run already started or finished, skip it this pass; otherwise append `## Execution Log\nStarted: {timestamp}` immediately.

Do exactly what the card's "Once approved" section describes — nothing broader. When genuinely done: append what was posted (with links) to the Execution Log, then `notion-update-page` → `update_properties` → `"Status": "Done"`. If something in the card is blocked or unclear, leave a note and keep Status as "In progress" rather than guessing.

## Standing guardrails — apply regardless of which job

- **Never post to any platform other than Instagram** unless a specific approved Notion card explicitly authorizes it.
- **Never reference customer testimonials, results, or social proof** — BeatPilot has zero verified paying customers.
- **Never reference specific prices** unless confirmed in `BeatPilot_Product_Offerings.docx`.
- **Never use the retired blue-accent branding** — everything should match "High Voltage" (lime + violet).
- **Never touch a card where `Social Task` is unchecked**, and never touch `CEO Task` cards — that's `ceo-executor`'s lane, not this skill's.
- **Never set `Status` to "Done"** unless the content was actually posted — a blocked or partial attempt stays "In progress" with a clear note.

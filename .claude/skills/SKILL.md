---
name: beatpilot-blog-engine
description: Research, draft, review, visualize, publish, and promote BeatPilot blog content using the approved Google Sheets pipeline and Google Drive workspace. Use this skill whenever the user asks to find blog topics, research competitors or RSS feeds, write or review a BeatPilot article, plan or generate blog visuals, prepare a post for publishing, build a social promotion campaign for a published post, or audit/refresh existing BeatPilot blog content. Also trigger for "run the blog routine," "run the BeatPilot blog engine," or any request that touches the Beat Pilot Blog Google Drive folder or the BeatPilot Blog Content Pipeline sheet.
---

# BeatPilot Blog Engine

## Purpose

Operate the full BeatPilot content lifecycle: research, approved-article production, visual planning, publishing preparation, social distribution, and content refresh.

All BeatPilot blog files must be housed in the Google Drive folder:

`Beat Pilot Blog`

The authoritative content approval queue is the Google Sheet:

`BeatPilot Blog Content Pipeline` (inside `Beat Pilot Blog / 01 Topic Pipeline`)

Brand and product source-of-truth docs live in `Beat Pilot Blog / 08 Brand and Product Information`:
`BeatPilot Brand Guide`, `BeatPilot Product Information`, `BeatPilot Pricing and Trial Information`, `BeatPilot CTA Library`, `BeatPilot Blog Visual System`, `BeatPilot Blog Editorial Rules`. Always check these before writing or generating anything customer-facing — they are updated periodically and older cached knowledge (including anything from a previous conversation or this skill's own training) may be stale. If more than one copy of any of these exists in the folder, use the most recently modified one and flag the duplicate to the user rather than guessing which is current.

## Mandatory rules

1. Never write an automatically discovered topic's status as `Approved`. Discovered ideas may only be written as `Idea` or `Research`.
2. Never mark a topic `Approved` — that status change is human-only, made directly in the Sheet.
3. Never copy or lightly rewrite competitor content. Study structure and gaps, not phrasing.
4. Never fabricate statistics, research, screenshots, or product capabilities. If a fact isn't confirmed in `BeatPilot Product Information` or `BeatPilot Pricing and Trial Information`, flag it — don't guess.
5. Always verify current BeatPilot pricing, trial terms, features, and screenshots against the docs in `08 Brand and Product Information` before publishing anything that references them.
6. Always check for duplicate topics and keyword cannibalization against existing Sheet rows and `01 Topic Pipeline / Published Topic Log` before adding a new idea.
7. Never publish or schedule social posts without explicit human approval in the current conversation.
8. Store all outputs inside the `Beat Pilot Blog` Google Drive structure — never leave a draft, brief, or asset only in the local workspace.
9. Preserve manual edits. Never silently overwrite a human-edited Sheet row, Doc, or Drive file — if a rewrite is needed, say so and confirm first.
10. Report failed or incomplete steps honestly. If a mode can't complete (missing data, ambiguous approval status, tool failure), stop and say what happened rather than filling the gap with a guess.

## Operating modes

This skill has six operating modes. Read the matching file in `references/` before executing that mode — don't rely on this summary alone, the reference files contain the actual procedure.

1. **Research Mode** → `references/research-mode.md`
2. **Approved Article Mode** → `references/article-mode.md`
3. **Visual Production Mode** → `references/visual-mode.md`
4. **Publishing Preparation Mode** → `references/publishing-mode.md`
5. **Social Campaign Mode** → `references/social-mode.md`
6. **Content Refresh Mode** → `references/refresh-mode.md`

## Routine selection

Use `references/research-mode.md` when the user asks to:
- Find blog topics
- Research competitors
- Analyze RSS feeds
- Find content gaps
- Perform keyword research
- Add ideas to the Google Sheet

Use `references/article-mode.md` when the user asks to:
- Create the next approved article
- Produce a research brief
- Write an article draft
- Generate an SEO package

Use `references/visual-mode.md` when the user asks to:
- Generate article images
- Design charts or infographics
- Create a 4:5 cover
- Create a 9:16 explainer

Use `references/publishing-mode.md` when the user asks to:
- Prepare an approved article for publishing
- Validate SEO and structured data
- Send the publishing package to Replit

Use `references/social-mode.md` when the user asks to:
- Create captions
- Create UTM links
- Find available posting slots
- Prepare a Blotato campaign

Use `references/refresh-mode.md` when the user asks to:
- Audit published articles
- Find outdated content
- Improve underperforming content

If a request spans more than one mode (e.g. "write and publish the next article"), work through the modes in the order they appear in the Full Routine below, stopping at each required approval gate.

## Full routine

When asked to run the full BeatPilot blog routine:

1. Verify current BeatPilot information (pricing, features, screenshots) against `08 Brand and Product Information`.
2. Sync the topic Sheet — check for stale entries, duplicates, and anything a human already edited.
3. Check approved sources for research (see `references/research-mode.md`).
4. Research and score new topics.
5. Add qualified ideas as `Idea` or `Research` — never `Approved`.
6. Check for human-approved topics (`Status = Approved` in the Sheet).
7. Create the next approved article's package (research brief → outline → draft → SEO package → internal links → CTA plan → visual plan).
8. **Stop for article approval.** Do not proceed to visuals without a human confirming the draft.
9. After approval, create and validate visuals.
10. **Stop for publishing approval.** Do not send anything to Replit without confirmation.
11. After publication, create the social campaign.
12. **Stop for social approval.** Do not schedule or post without confirmation.
13. After approval, send the campaign to Blotato.
14. Save all records and logs — update the Sheet's Status/Date columns and any relevant folder location for every asset produced this run.

Never skip a stop-for-approval step even if the user seems to be requesting the full routine end-to-end in one message — approval gates exist specifically so a human sees the work before it goes anywhere public.

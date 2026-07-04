---
name: ceo-executor
description: Execute BeatPilot growth tasks that the CEO Advisor skill suggested and the user approved by moving to "In progress" in the Beat Pilot Notion board, then move the card to "Done" when finished. Only acts on cards where the "CEO Task" checkbox is checked — never touches other project cards on the board. Use when asked to run the CEO executor, check for approved CEO tasks, or process the Beat Pilot board.
---

# CEO Executor (BeatPilot)

Carry out a CEO-suggested task, but only once a human has approved it by moving the card to "In progress." This is the execution half of `ceo-advisor`; it never invents new suggestions and never processes a card that skill didn't create.

## Find eligible tasks

Query the Beat Pilot data source directly rather than guessing:

notion-query-data-sources (sql mode)
data_source_urls: ["collection://3932ee00-89fb-800a-a8e6-000bf3b9f20d"]
query: SELECT * FROM "collection://3932ee00-89fb-800a-a8e6-000bf3b9f20d" WHERE "Status" = 'In progress' AND "CEO Task" = '__YES__'

Only rows matching **both** conditions are in scope. A human-created card that happens to be "In progress" (e.g. "Public launch of iOS app," "Quarterly sales planning") is never touched, because `CEO Task` won't be checked on it.

For each eligible row, fetch the full page to read the description `ceo-advisor` wrote: Recommended action, Suggested steps, and the "Executable by Claude?" line.

## Avoid double-executing

Before starting, check the page content for an "## Execution Log" section. If one exists and shows a run already completed or in progress, skip that card this pass. Otherwise, immediately `insert_content` at the end of the page:

## Execution Log
Started: {timestamp}

This is the guard against a later poll picking up the same card mid-run.

## Decide how to execute

Read the card's "Executable by Claude?" line:

- If it names a specific skill (ad-remixer, motion-design, client-finder) or a clear content task (draft a blog post, build a spreadsheet, write outreach copy), do that work using the matching existing skill or tool.
- If it says "No," or the task requires a real-world action only the user can take (spending money, hiring, an external signup), do not attempt it. Append a note explaining why it can't be auto-executed and leave `Status` as "In progress" — never mark work "Done" that wasn't actually done.
- If it's ambiguous, do the parts that are genuinely executable (research, a draft) and clearly flag in the log what still needs the user's input, again without marking it Done.

## Scope of what this skill may touch

**May use**: Google Drive (read/write within BeatPilot folders, including appending to the CEO Journal), Higgsfield, Blotato (drafting/scheduling BeatPilot's own Instagram content), web search, document-creation skills (docx/pptx/xlsx), and the Notion Beat Pilot board.

**Read-only, never act on**: Stripe and Replit. `ceo-advisor` uses these to observe revenue and app health, but this skill must never write to Stripe (no refunds, no price/product changes, no `stripe_api_write`) and must never modify or redeploy the Replit app. If a card's recommended action would require changing pricing, issuing a refund, or shipping a code change to the live app, treat it as **not** executable — append a note saying it needs the user to do it directly, and leave Status as "In progress."

**Must never use**: any billing/payment write action, website deploy/delete/secrets tools, or anything outside the BeatPilot Drive/Instagram/Notion scope — even if those connectors happen to be available in the workspace.

## Finishing a task

When the work is genuinely complete:

1. Append a closing note to the Execution Log: what was produced, with links (Drive file, Blotato post, generated asset URL).
2. `notion-update-page`, command `update_properties`, set `"Status": "Done"`.
3. Leave `CEO Task` checked so there's a clean audit trail of what the CEO system produced and shipped.

## Guardrails

- Never touch a card where `CEO Task` is unchecked, regardless of its Status.
- Never set `Status` to "Done" unless the described work was actually produced — partial or blocked work stays "In progress" with a clear note explaining what's outstanding.
- Never delete or overwrite existing page content; only append to the Execution Log.
- If a task looks like it could affect real customers, spend, or public-facing brand claims in a way the original suggestion didn't anticipate, stop and leave a note asking the user to confirm before continuing.

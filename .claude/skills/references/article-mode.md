# Approved Article Mode

Use this mode to turn a human-approved topic into a finished, reviewed article draft. This mode never approves anything — it only produces work for a human to review.

## 1. Select only Approved rows

Query `BeatPilot Blog Content Pipeline` for rows where `Status = Approved`. If none exist, stop and tell the user — do not fall back to an `Idea` or `Research` row, even if it looks ready.

If more than one `Approved` row exists, ask the user which to produce next, or default to the oldest `Date Added` / highest `Opportunity Score` if they ask you to just pick one — state which rule you used.

## 2. Create the research brief

Use `templates/research-brief.md`. Fill in every field using:
- The Sheet row's existing data (Topic, Proposed Title, Primary Keyword, clusters, Search Intent, Buyer Stage, etc.)
- Fresh research into current search results and competitors for this specific topic (see Research Mode's competitive-research steps — this is the same discipline, just deeper for one topic).
- `BeatPilot Product Information` for the Relevant BeatPilot Feature and BeatPilot Content Gap fields — don't guess which feature applies.

Save the completed brief to `03 Research Briefs / In Progress`, then move it to `03 Research Briefs / Approved` once you're confident it's ready to write from (this is a self-check, not a human approval gate — the human approval gate is the article draft itself, in step 8).

## 3. Create an outline

Build a structure from the brief's "Recommended Article Structure" and "Questions the Article Must Answer" fields. Every section should map to a real reader question — don't pad with generic filler sections.

## 4. Write the article

Follow `BeatPilot Blog Editorial Rules` exactly:
- Lead with the real producer pain point (their words, not ours) before introducing BeatPilot.
- Use the assigned messaging hooks from `BeatPilot CTA Library` where appropriate — don't rephrase them.
- Never state a guaranteed outcome, never cite an unverified stat, never reference a reader's personal hardship to create urgency.
- Use "niche" (not "lane") in the article body.
- Keep paragraphs short and scannable.
- Every specific product claim must be checked against `BeatPilot Product Information` and `BeatPilot Pricing and Trial Information` before it goes in the draft.

## 5. Generate the SEO package

- Meta title (reflecting the Primary Keyword and Search Intent)
- Meta description (under ~160 characters, includes Primary Keyword)
- URL slug
- Suggested header structure (H1/H2/H3) matching the outline

## 6. Generate internal links

Check `04 Article Drafts / Published` for existing live articles that genuinely relate to this topic's cluster, and link to them naturally. Don't force a link where the connection is weak.

## 7. Generate the CTA plan

Pick the CTA from `BeatPilot CTA Library` that matches this post's topic AND funnel stage (see the CTA Library's topic-matching and funnel-stage sections) — don't default to the general trial CTA if a more specific one fits.

## 8. Generate the visual plan

Don't generate visuals yet — that's Visual Production Mode's job. Instead, produce a visual plan (using `templates/visual-plan.md`) listing what's needed: featured image concept, whether a real product screenshot is required (and which one, from Product Information's Current Screenshot Filename field), any charts/infographics, and a 4:5 cover concept.

## 9. Change status to Review

Once the draft, SEO package, internal links, CTA plan, and visual plan are complete:
- Save the full article package (use `templates/article-package.md`) to `04 Article Drafts / Drafting`, then move it to `04 Article Drafts / Review`.
- Update the Sheet row's Status to `Review` and set Last Reviewed to today's date.

**Stop here.** Do not proceed to Visual Production Mode or Publishing Preparation Mode until a human has reviewed the draft and moved it (or told you to move it) to `04 Article Drafts / Approved`.

# Publishing Preparation Mode

Use this mode only after both the article draft (`04 Article Drafts / Approved`) and its visuals have been explicitly human-approved. This mode prepares the package — it does not itself constitute publishing approval, and it must stop before actually pushing anything live.

## 1. Validate article approval

Confirm the article package is physically in `04 Article Drafts / Approved`, not just written. If it's still sitting in `Drafting` or `Review`, stop and tell the user — do not proceed on the assumption that "it's probably fine."

## 2. Validate visuals

Confirm every visual referenced in the article's visual plan actually exists in the correct `05 Blog Images` subfolder and was approved in Visual Production Mode. Flag any missing or unapproved asset rather than publishing with a placeholder.

## 3. Build the final publishing package

Compile, using `templates/article-package.md` as the base:
- Final article body (Markdown or HTML, matching whatever the CMS/Replit site expects)
- Meta title, meta description, URL slug
- Open Graph image reference
- Structured data (JSON-LD) — Article or BlogPosting schema, matching the pattern already used on beatpilot.io per its SEO setup (WebSite + Organization + SoftwareApplication @graph schema is the site's existing pattern; extend consistently rather than inventing a new schema shape)
- Internal links list
- CTA (exact copy + link, from the CTA Library)
- Primary/supporting keywords, for tracking

Validate before sending:
- No unconfirmed stats or claims slipped in during drafting (recheck against `BeatPilot Product Information` / `BeatPilot Pricing and Trial Information` one more time — these can change between draft and publish).
- No ALL CAPS full headlines, no lime used as a general accent, no green — quick pass against `BeatPilot Blog Editorial Rules`.
- Meta description length and title length are within normal SEO bounds.

## 4. Send to Replit

If a Replit connector/tool is available in this session, use it to either ask a question about the current blog implementation (routing, CMS shape) or submit the update via `update_app_using_prompt` — but confirm with the user first exactly what "send to Replit" should do in their current setup (direct file commit vs. a prompt-driven update) rather than assuming. If no Replit tool is available, hand the user the complete package and clear instructions for where it goes instead.

## 5. Update Google Sheets after successful publication

Only after publishing is confirmed successful:
- Set Status to `Published`
- Set Publish Date
- Set Published URL
- Move the article package from `04 Article Drafts / Approved` to `04 Article Drafts / Published`
- Log the run in `10 System Logs / Publishing Logs`

If publishing fails or is only partially successful, do not mark the row `Published` — report exactly what happened and what state things were left in.

**Stop here.** Do not proceed to Social Campaign Mode until the human confirms the post is actually live and ready to promote.

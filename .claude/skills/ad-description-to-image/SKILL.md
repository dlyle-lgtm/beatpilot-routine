---
name: ad-description-to-image
description: >
  Turn one finished ad description doc from BeatPilot's "Ad Idea Docs" Google Drive folder into
  an actual static 9:16 ad image, generated with Higgsfield, and save it into the "Make Brand
  Packaging" Drive folder so it's ready for the brand-packaging skill to turn into a full
  cross-platform posting set. Never reuses a doc that's already been turned into an image.
  Trigger this skill whenever the user asks to "turn the ad idea doc into an image," "generate the
  next ad from the idea docs," "make the ad from that description," "run the ad image skill," or
  asks to produce an actual ad image/creative from something in the Ad Idea Docs folder — even if
  they don't say the word "skill." Also trigger for "do the next one" / "keep going" follow-ups
  once this skill has already run once in the conversation. This is the natural second step after
  the ad-reference-to-description skill, which writes the docs this skill consumes.
---

# Ad Idea Doc → 9:16 Ad Image

You're taking a fully-written ad description (produced by the ad-reference-to-description skill,
or written by hand) and actually building the image from it — a single static 9:16 ad, generated
with Higgsfield. The doc already contains every decision that matters (copy, screenshot choice,
logo placement, CTA); your job here is faithful execution, not re-deciding the creative.

This skill processes **one doc per run**. If the user wants several, repeat the whole flow (Steps
1-5) once per doc, picking the next unused one each time.

---

## Step 1 — Pick the next unused ad idea doc

**Ad Idea Docs folder:** `18vIvCdSV5OKvENlAcMK1fUsAJ6CZ-ObG`
**Ad-Idea-Docs-Used-Log folder:** `1P12LsOJ7H5vIdJUmcEbHCrf4sIh7G_v5`

Same dedup pattern as the rest of the content pipeline (Producer Tips carousels, ad references): a
marker file per used doc, dropped in the used-log folder, since Drive has no native "used" tag.

1. List files in the Ad-Idea-Docs-Used-Log folder (`Google Drive:search_files` with
   `parentId = '1P12LsOJ7H5vIdJUmcEbHCrf4sIh7G_v5'`). Read each marker to collect the set of source
   doc file IDs already used.
2. List files in the Ad Idea Docs folder, sorted oldest-first by `createdTime` — the earliest
   doc that hasn't been used yet is "next."
3. Walk the list and pick the first file ID **not** in the used set.
4. If every doc has already been used, tell the user the folder is exhausted and ask whether to
   wait for the ad-reference-to-description skill to add more, or stop here.

Read the full doc content (`Google Drive:read_file_content`) before moving on.

---

## Step 2 — Extract everything needed to build the image

The doc (if it came from ad-reference-to-description) is already structured section-by-section.
Pull out, precisely:

- **Exact on-screen copy** for every section (headline, subhead, body, supporting lines) — use it
  verbatim, don't paraphrase or "improve" it. The doc already made those calls.
- **Which BeatPilot screenshot to use**, by filename, if the doc specifies one. Confirm it still
  exists in the BeatPilot-Screenshots folder (`1XS-a82qXbLJsYynZIg2W0CpREW3PLXDX` —
  `Google Drive:search_files` for the exact filename) before relying on it. If it's missing or the
  doc didn't name one, note that and either pick the closest still-available match or describe the
  visual content generically per the doc's own fallback description.
- **Logo requirement** — every BeatPilot ad needs the real logo. Use the Higgsfield media ID
  `a76ac00c-c172-43c0-b890-4c31ae874f7b` (already-uploaded BeatPilot logo) as a reference image.
- **Website URL** — `www.BeatPilot.io` must appear; confirm the doc specifies where and how.
- **CTA copy** — exact wording and placement.
- **Visual treatment notes** — background, color, card/framing style, mood — everything the doc
  specifies about how it should look, not just what it says.

If the doc is missing something essential (no CTA specified, no copy for a section, ambiguous
layout), don't invent it silently — flag the gap to the user and ask, or make the most reasonable
inference and say what you assumed.

---

## Step 3 — Generate the image

Use `Higgsfield:generate_image`, `aspect_ratio: "9:16"`, `model: "nano_banana_2"` (this is what the
rest of BeatPilot's ad/carousel creative already uses — stay consistent unless the doc calls for
something the model can't do well, e.g. very dense multi-column text, in which case
`nano_banana_pro` is the fallback for text-heavy density).

Build one prompt that folds in everything from Step 2: the exact copy for each section top-to-
bottom, the background/color treatment, where the screenshot sits and how it's framed, where the
logo sits, where the URL and CTA sit. Write it the same way the established BeatPilot ad prompts
are written — explicit section-by-section spatial instructions, not a vague mood description.

**References:**
- Always pass the logo (`a76ac00c-c172-43c0-b890-4c31ae874f7b`) as a reference image, and instruct
  the model to place it pixel-identical, unmodified.
- If a real product screenshot is also called for, you can try passing it as a second reference
  alongside the logo — but this has been unreliable in practice (multi-reference calls to
  nano_banana_2 have failed outright rather than degrading gracefully). If a two-reference call
  errors, retry with **just the logo** as the reference and describe the screenshot's actual
  content in the prompt text instead (numbers, labels, layout) so the model recreates it visually
  rather than compositing the real file. Don't burn more than one retry on this — fall back
  promptly rather than looping.
- To get a Higgsfield media ID for a screenshot that hasn't been uploaded to Higgsfield before,
  you'll need to get the file into Higgsfield's system first (check `Higgsfield:media_import_url`
  against the Drive file's direct-download link, or upload flow) before it can be used as a
  `medias` reference — it can't be referenced by Drive file ID directly.

Call `Higgsfield:job_display` on the result to confirm it completed and get the final image URL.

---

## Step 4 — Save the image to Drive

**Make Brand Packaging folder:** `1NTMeaPaq2SJi0HfKtjUVWo799MHQQQkj`

Drive's `create_file` can't ingest an external URL directly, so:
1. Download the generated image's bytes from its Higgsfield CDN URL (`bash_tool` + `curl`, or
   Python).
2. Base64-encode the bytes.
3. Call `Google Drive:create_file` with `contentMimeType: "image/png"`,
   `disableConversionToGoogleType: true`, `base64Content` set to the encoded bytes, `parentId`
   `1NTMeaPaq2SJi0HfKtjUVWo799MHQQQkj`, and a clear `title` (e.g. the ad idea doc's short name plus
   today's date).

This folder is the same intake folder the **brand-packaging** skill watches — dropping the image
here means it's immediately ready to be picked up and turned into every ratio/platform variant and
scheduled via Blotato, the next time that skill runs. This skill only produces and saves the
master 9:16 image; it doesn't trigger brand-packaging itself.

---

## Step 5 — Log usage and wrap up

1. Create a marker file in Ad-Idea-Docs-Used-Log (`1P12LsOJ7H5vIdJUmcEbHCrf4sIh7G_v5`) with at
   minimum:
   - Source ad idea doc's file ID and title
   - Date used
   - The Higgsfield job ID and the new Drive image file's ID/link
2. Show the user the generated image (`Higgsfield:job_display`) and tell them which doc it came
   from and where it landed in Drive.
3. If they say "next one" / "keep going," repeat from Step 1.

---

## Notes & Rules

- **One doc, one image, per run.** Don't batch multiple ad idea docs into one generation.
- **Never skip the used-log.** Every image produced needs a matching marker file, or the doc could
  get reprocessed later.
- **Faithful execution, not re-creativity.** The doc already made the creative decisions (copy,
  screenshot, layout). Don't rewrite copy or change the concept — your job is turning the written
  description into pixels as accurately as possible.
- **Real logo only, always as a reference image** — never let the model redraw or reinterpret it.
- **Real screenshots preferred over invented ones**, but don't let an unreliable multi-reference
  API call block the whole run — fall back to describing the screenshot's content in text after
  one failed attempt, per Step 3.
- **URL and CTA are mandatory** — if the doc specifies them (it should), they must appear in the
  final image exactly as written.
- **This skill hands off to brand-packaging**, not the other way — dropping output in Make Brand
  Packaging is the intended bridge between the two, but don't invoke brand-packaging yourself
  unless the user explicitly asks you to continue the pipeline that far in the same run.
- **Tool names:** `Google Drive:search_files`, `Google Drive:read_file_content`,
  `Google Drive:create_file`, `Higgsfield:generate_image`, `Higgsfield:job_display`,
  `Higgsfield:media_import_url` (only if uploading a new screenshot reference to Higgsfield),
  `bash_tool` (download + base64-encode the generated image before re-uploading to Drive).
- **Known risk — network egress.** Downloading the Higgsfield CDN image via `bash_tool`/`curl`
  depends on that host being reachable from the sandbox. This has been an open issue elsewhere in
  the BeatPilot pipeline (the brand-packaging skill hit the same wall with Higgsfield/ffmpeg
  egress). If the download fails with a network/allowlist error, don't silently give up or fake
  success — tell the user directly that the image generated correctly but couldn't be saved to
  Drive due to sandbox network restrictions, and that the host (e.g. `*.cloudfront.net`) may need
  to be added to the environment's network egress settings. Only log the doc as used once the
  image is actually confirmed saved in Make Brand Packaging — don't mark a doc "used" for a run
  that generated an image but failed to save it, or the doc would be lost without ever producing
  usable output.
- **Language:** match the user's language throughout.

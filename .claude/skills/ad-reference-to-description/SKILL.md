---
name: ad-reference-to-description
description: >
  Study one ad from BeatPilot's "Ad References" Google Drive folder, figure out what makes it
  work (hook, structure, visual style, why it converts), and write a fully-detailed creative
  description of a BeatPilot version of that ad — a 9:16 vertical ad that tells producers why
  they should use BeatPilot. The description is written as a new Google Doc in the "Ad Idea
  Docs" folder, one doc per reference ad, and never reuses a reference that's already been
  processed. Trigger this skill whenever the user asks to "run the ad reference skill," "turn an
  ad reference into an ad idea," "write up the next ad idea doc," "go through the ad references
  folder," or asks for a BeatPilot ad concept/description based on a reference ad in Drive — even
  if they don't say the word "skill." Also trigger for "do the next one" / "keep going" follow-ups
  once this skill has already run once in the conversation.
---

# Ad Reference → BeatPilot Ad Description

You're turning one reference ad (something that's presumably working well for another brand) into
a fully-written creative brief for a BeatPilot version — a 9:16 ad explaining why producers should
use BeatPilot. You don't generate an image here; you write the single best, most detailed
description of the ad someone would need in order to go build it. Be thorough — this doc is the
only thing standing between "here's a good idea" and someone actually being able to produce the ad
from it.

This skill processes **one reference ad per run**. If the user wants several, repeat the whole
flow (Steps 1-5) once per ad, picking the next unused reference each time — don't batch multiple
ads' worth of analysis into one doc.

---

## Step 1 — Pick the next unused reference ad

**Ad References folder:** `1rnNWZBUuWm2k-mDvWjdyRkqHMC4SLimY`
**Ad-References-Used-Log folder:** `1QT0QjCyWQ1-fZWPYqsK8pBc3HhS8h5Bk`

The "Ad References" folder fills up with screenshots/photos of ads pulled from wherever the user
finds inspiration. Since Drive has no easy way to tag a file as "used," tracking happens the same
way the carousel content pipeline does it: a marker file per used reference, dropped in the
Ad-References-Used-Log folder.

1. List files in the Ad-References-Used-Log folder (`Google Drive:search_files` with
   `parentId = '1QT0QjCyWQ1-fZWPYqsK8pBc3HhS8h5Bk'`). Read each marker to collect the set of source
   file IDs already used.
2. List files in the Ad References folder, sorted oldest-first by `createdTime` (treat this as
   "first available" — the earliest-added reference that hasn't been used yet).
3. Walk the list in that order and pick the first file ID **not** present in the used set. That's
   your reference ad for this run.
4. If every file in the folder has already been used, tell the user the folder is exhausted and
   ask whether to start over from the oldest one, treat only-newer-uploads as fair game going
   forward, or stop here. Don't silently reprocess a used ad.

View the chosen image directly (`view`, or read its content snippet from search results if it's
already OCR'd) before moving on — you need to actually look at it, not just know its filename.

**If the file clearly isn't an ad** — a personal screenshot, a meme, a random photo with no
brand/product being promoted, nothing resembling a hook-to-CTA structure — don't force an analysis
out of it. Log it in the used-log with a note (`Status: skipped — not an ad`) so it's never picked
again, then move to the next file in the order automatically. Mention at the end which file(s) got
skipped this way and why, but don't stop and ask permission for each one — only pause and ask the
user if a file is a genuine judgment call (e.g. it's ambiguous whether it's an ad or just stylized
content) rather than clear-cut noise.

---

## Step 2 — Study the reference ad

Look closely and write out, for your own use (a short internal breakdown, not the final doc yet):

- **The hook** — the first thing the eye reads or sees, and why it stops the scroll
- **Structure/flow** — the order information appears in, top to bottom, and how it builds toward
  the CTA
- **What's actually being sold and how** — which claims, proof points, or visuals do the
  persuading, and in what order
- **Visual style** — color, typography, photography vs. UI-screenshot vs. illustration, mood,
  pacing (especially important for 9:16 — is it one static idea or does it read like a sequence?)
- **CTA** — what action it asks for and how it's framed
- **Why this is working** — your actual read on why this ad converts. This is the part that
  matters most: don't just describe the ad, diagnose it. Is it the specificity of the numbers?
  The relatability of the pain point? The visual proof over the claim? Say so explicitly, because
  Step 3 needs to preserve *that mechanism*, not just the surface layout.

---

## Step 3 — Learn what to actually advertise

Before writing anything BeatPilot-specific, ground yourself in the real product so nothing in the
final description is invented or generic:

- **BeatPilot-Info folder:** `1KF05eVO3FVotXQ5oD_KgnSRMTrjwEQ41` — read the docs here
  (`Google Drive:search_files` then `read_file_content`) for the actual feature set, pricing,
  target customer profile, and benefit language. Pull specific, concrete details (feature names,
  price points, the actual value proposition) rather than paraphrasing BeatPilot in the abstract.
- **BeatPilot-Screenshots folder:** `1XS-a82qXbLJsYynZIg2W0CpREW3PLXDX` — list what's in here.
  You'll pick specific screenshot(s) from this folder in Step 4 to serve as the ad's visual proof,
  the same way the reference ad likely used a real product screenshot rather than an abstract
  mockup. Note filenames so the doc can reference them exactly.
- **Copy rule — no invented results.** BeatPilot doesn't have confirmed paying customers yet, so
  the description must stick to pain point + product promise only. Don't write copy implying
  specific outcomes, dollar amounts earned, or social proof ("producers are already using this to
  make $X") — those aren't true yet and shouldn't appear in the brief.

---

## Step 4 — Write the full ad description

This is the core of the skill: translate what you learned in Step 2 into a BeatPilot-specific
version, using what you learned in Step 3. Think of yourself as writing the creative brief you'd
hand to a designer or an image-generation model — someone with zero context should be able to
build the exact ad from this doc alone.

**Format:** 9:16 vertical, matching the reference ad's underlying persuasive mechanism (not
necessarily its literal layout — see below).

**Fidelity call:** keep whatever made the reference ad work (per your Step 2 diagnosis), but don't
mechanically clone its structure if a different structure would sell BeatPilot better to a
producer audience. You're building the highest-converting BeatPilot ad you can, using the
reference as inspiration and proof-of-what-works, not as a template to trace.

The doc must cover, in this order:

1. **One-line concept** — what this ad is and why it'll work for BeatPilot, in a sentence.
2. **What we're borrowing from the reference and why** — 2-4 bullets naming the specific
   mechanism(s) from Step 2 you're keeping (e.g. "opens on a relatable frustration before showing
   the fix," "uses a real dashboard screenshot as proof instead of a claim") and why each one maps
   well onto BeatPilot's audience.
3. **Full shot-by-shot / section-by-section description** — walk down the 9:16 frame top to
   bottom (or beat to beat, if it's a sequence). For each section specify:
   - Exact on-screen text (headline, subhead, body copy) — write the actual copy, not a
     placeholder
   - Exactly which screenshot from BeatPilot-Screenshots to use here (by filename), or "no
     screenshot — [describe what's shown instead]" if that section doesn't use one
   - Visual treatment (background, color, how the screenshot is framed/cropped/overlaid, motion
     notes if it's meant to be an animated/video treatment)
4. **Logo placement** — every BeatPilot ad needs the real logo somewhere in it. Say exactly where
   (e.g. "top center, moderate size, clear space around it") and reference the logo file:
   `15QM69E47nx5Z3g57DexEn7Gc2rYsLFzA`. Never describe inventing a new logo/wordmark — this one is
   real and must be used as-is.
5. **Website URL placement** — `www.BeatPilot.io` must appear visibly somewhere on the ad. Say
   exactly where and how it's styled.
6. **CTA** — the specific call to action, matched to what actually works for BeatPilot right now
   (e.g. a free trial framing, or "Comment FLSTUDIO for the link" if this is going out as an
   Instagram/TikTok-style post rather than a paid ad unit — infer which from context, or ask if
   genuinely unclear).
7. **Why this converts** — close with a short paragraph making the explicit case for why this ad,
   for this audience, will perform — tying back to the mechanism identified in Step 2 and the real
   product substance from Step 3. This isn't marketing fluff for its own sake; it's the actual
   argument for why the creative choices above are the right ones.

Don't hedge or offer multiple alternative versions in one doc — commit to one strong, fully
detailed concept. If you genuinely see two very different directions worth capturing, that's a
signal to treat them as separate runs against separate reference ads, not two options in one doc.

---

## Step 5 — Create the doc and log usage

1. **Ad Idea Docs folder:** `18vIvCdSV5OKvENlAcMK1fUsAJ6CZ-ObG` — create one new Google Doc here
   (`Google Drive:create_file`, plain text content is fine — Drive will offer to convert it, or
   pass `disableConversionToGoogleType: true` if the user prefers a flat `.txt`). Title it
   something identifiable, e.g. `Ad Idea - [short description of the reference ad] - [today's
   date]` — not a generic "Ad Idea Doc 1," since these accumulate over time and need to stay
   distinguishable at a glance.
2. Write the full Step 4 description as the doc's content. One reference ad, one doc — never
   append multiple ad ideas into the same file.
3. **Log the reference as used.** Create a marker file in the Ad-References-Used-Log folder
   (`1QT0QjCyWQ1-fZWPYqsK8pBc3HhS8h5Bk`), named after the source file's ID or a short slug, with
   at minimum:
   - Source ad file ID and filename
   - Date used
   - The new Ad Idea Doc's file ID/link
   This is what Step 1 checks on the next run — skipping it means the same reference ad could get
   reprocessed later.
4. Tell the user which reference ad you used and share the new doc's link. If they say "next one"
   / "keep going," repeat from Step 1 — Step 1 will naturally pick the next unused file since this
   one's now logged.

---

## Notes & Rules

- **One ad, one doc, every time.** Never combine multiple reference ads into a single Ad Idea doc,
  and never write more than one concept into a doc for a single reference ad.
- **Never skip the used-log.** Every run that produces a doc must also produce a matching marker
  file — this is the only thing preventing duplicate work down the line.
- **Real screenshots only.** Always pick from the actual files in BeatPilot-Screenshots by
  filename — never invent a fictional dashboard/UI description as a substitute when real ones
  exist.
- **Real logo only.** Reference the actual logo file; never describe generating a new one.
- **URL is mandatory.** `www.BeatPilot.io` must be specified somewhere in every description.
- **No invented results/social proof** in any copy — pain point and product promise only, per
  Step 3.
- **Conversion quality over fidelity.** The reference ad is inspiration for a *mechanism* that
  works, not a template to trace line-for-line. If the audience or product calls for a different
  structure, say so and take it — same principle as the ad-remixer skill.
- **Drive tools:** `Google Drive:search_files` (list/filter), `Google Drive:read_file_content`
  (BeatPilot-Info docs, natural-language read), `Google Drive:create_file` (both the Ad Idea doc
  and the used-log marker file), `view` for looking at reference ad images directly if needed.
- **Language:** match the user's language throughout, including the doc's content.

---
name: gpt-image-2
description: Generate or edit images with OpenAI's GPT Image 2 via OpenRouter. Best-in-class for fine typography — chalkboards, signs, menus, posters, packaging mockups, UI screenshots, anything with legible text baked into the image. Single POST to OpenRouter's dedicated Image API, base64 response, no polling. Triggers on gpt image 2, gpt-image-2, chatgpt image, openai image, typography image, sign mockup, poster mockup, menu image, packaging mockup, legible text in image, monthly report infographic labels, ad headline image.
---

# GPT Image 2 (via OpenRouter)

OpenAI's strongest model for **legible text inside images**. Use this when the picture *is* the typography — chalkboards, signage, menu boards, packaging mockups, UI screenshots, book covers, ad headlines, and BeatPilot's monthly report infographic panels (heatmap labels, leaderboard headers) where Nano Banana / Higgsfield tends to mangle rendered text.

For photoreal scenes, style-consistent brand imagery, or anything that doesn't need crisp in-image text, keep using Higgsfield's Nano Banana / Soul ID pipeline — that's still the better tool for those jobs.

| Field | Value |
|-------|-------|
| Model ID | `openai/gpt-image-2` |
| Provider | OpenRouter (dedicated Image API — not chat completions) |
| Endpoint | `POST https://openrouter.ai/api/v1/images` |
| Method | Sync — single POST returns base64 image data, no polling |
| API Key | `.env` → `OPENROUTER_API_KEY` |
| Docs | https://openrouter.ai/docs/guides/overview/multimodal/image-generation |
| Billing | All-or-nothing — failed/cancelled generations are never billed |

---

## Setup

1. Use your existing OpenRouter account and API key.
2. Add to `.env`:
   ```
   OPENROUTER_API_KEY=your_key_here
   ```
3. No SDK required — plain `curl`/`fetch`/`requests`.

---

## How to call it

Auth header is standard Bearer, same as any other OpenRouter call:

```
Authorization: Bearer $OPENROUTER_API_KEY
```

### Minimum request

```bash
curl -X POST "https://openrouter.ai/api/v1/images" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-image-2",
    "prompt": "A vintage diner chalkboard reading TODAY SPECIAL — Lobster Roll $24"
  }'
```

### Full request (recommended defaults for BeatPilot)

```json
{
  "model": "openai/gpt-image-2",
  "prompt": "...",
  "quality": "medium",
  "size": "1536x1024",
  "output_format": "png",
  "n": 1
}
```

### Response format

Images come back as **base64**, not a hosted URL:

```json
{
  "created": 1748372400,
  "data": [
    { "b64_json": "<base64-encoded-image>", "media_type": "image/png" }
  ],
  "usage": { "prompt_tokens": 0, "completion_tokens": 4175, "total_tokens": 4175, "cost": 0.04 }
}
```

Decode `b64_json` and write to disk (or upload to Drive) — don't assume a URL exists like the Fal/Higgsfield flows. `usage.cost` reports the actual billed amount per call, useful for tracking real spend against the report/ad pipelines.

### Image edit (reference image in, edited image out)

```json
{
  "model": "openai/gpt-image-2",
  "prompt": "Same layout, but change the headline to READ THE ROOM",
  "input_references": [
    { "type": "image_url", "image_url": { "url": "https://example.com/source.png" } }
  ]
}
```

`input_references` accepts HTTP(S) URLs or base64 data URLs. Describe the *change*, not the whole scene — same rule as before: this preserves more of the original than re-describing everything.

---

## Key Rules

1. **Default `quality: medium`** — `low`/`high` also available; reserve `high` for finals where typography legibility really matters (e.g. print-quality report exports).
2. **Response is base64, not a URL.** If you need to hand the result to another tool (Notion, Drive, a downstream Higgsfield reference call) that expects a URL, decode and upload it first — don't pass `b64_json` where a URL is expected.
3. **Billing is all-or-nothing.** A failed or cancelled generation is never charged — safe to iterate freely on prompts without worrying about wasted spend on failures (unlike partial-token billing on chat completions).
4. **No real transparent backgrounds unless the model/provider supports alpha.** `background: "transparent"` requires `output_format` of `png` or `webp` — otherwise you'll get a fake baked-in checkerboard.
5. **Multiple images per call:** pass `n` (up to 10) to get variants in one request — useful for the same "generate a few, pick one" pattern used with Higgsfield outputs.
6. **Aspect ratio / size:** use `size` (e.g. `"1536x1024"`, `"1024x1536"`) for explicit pixels, or `resolution` + `aspect_ratio` for tiered sizing (`"2K"` + `"9:16"` for Reel-style crops). Don't set both `size` (pixel form) and a conflicting `resolution`/`aspect_ratio` — that returns a 400.

---

## Prompting patterns

Same two patterns that work well for this model:

1. **Structured JSON prompts** for multi-element scenes (posters, infographics, comics, app mockups) — explicit slots (`type`, `subject`, `style`, `background`, `header`, `layout`) beat prose for complex layouts.
2. **Describe the change, not the scene** for edits — "same workers, but on phones now" beats re-describing everything.

For inspiration, the community-curated **awesome-gpt-image-2** library (https://github.com/YouMind-OpenLab/awesome-gpt-image-2, CC BY 4.0) has 700+ example prompts organized by category — fetch the README and adapt an example when unsure how to phrase a complex layout. Credit the original author if reusing a prompt verbatim.

---

## Where this fits in the BeatPilot pipeline

- **Monthly report infographic** — route text-bearing panels (upload heatmap labels, leaderboard headers, dashboard callouts) here instead of Nano Banana; keep photoreal/style panels on Higgsfield.
- **Ad creative headlines** — when `ad-description-to-image` needs a static ad with a headline/CTA baked into the image, this model renders that text far more reliably than Nano Banana.
- **Producer Tips / carousel title cards** — any slide where the "art" is mostly typography.

Keep everything else — photoreal scenes, Soul ID character consistency, style-matched brand imagery — on Higgsfield. This model is a specialist tool for the text-legibility gap, not a Higgsfield replacement.

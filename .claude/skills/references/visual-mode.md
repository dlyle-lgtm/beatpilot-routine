# Visual Production Mode

Use this mode only after an article has reached `04 Article Drafts / Approved` (or, for standalone visual requests, when the user explicitly asks for a specific asset). Nothing produced here goes live without a separate human approval — see the Approval requirement at the bottom.

## Design system note — check before using either name below

This mode's original spec refers to a "BeatPilot Light Intelligence design." That name isn't currently documented anywhere else in this project — the confirmed, current visual/brand system as of the last brand update is **"Neon Wheel v4"** (see `BeatPilot Brand Guide` and `BeatPilot Blog Visual System` in `08 Brand and Product Information`). If "Light Intelligence" refers to something distinct (a lighter/alternate mode, a specific illustration style, etc.), confirm with the user before assuming — don't invent a design language to match an unfamiliar name. Absent clarification, use Neon Wheel v4 as documented.

Core Neon Wheel v4 rules to hold to on every visual:
- Base surfaces: Near-Black #0B0E14, Charcoal #14181F, Off-White #F5F5F7 text.
- Five signal colors, each with exactly one meaning — Blue=new, Violet=premium, Magenta=rising, Orange=hot, **Lime=live/verified only** (never a general CTA/decorative color).
- Exactly one dominant gradient moment (Blue→Violet→Magenta→Orange) per visual — never a repeating pattern.
- Inter Display for on-image headlines, Inter for body/labels, JetBrains Mono for technical/numeric callouts.
- No robots, brains, circuit-board heads, holographic people, full-screen rainbow gradients, or AI-generated faces presented as real customers.

## OpenRouter image generation rules

For any image with legible text baked in (chalkboards, signage, menus, packaging mockups, UI screenshots, typography-heavy graphics, infographic labels), use the **gpt-image-2** skill (OpenRouter, `openai/gpt-image-2` via the dedicated Image API) — it's the reliable choice for rendered text, where other generators tend to mangle it.

For style/mood reference, character/scene generation, or reframing/outpainting existing assets, use Higgsfield per its existing confirmed media IDs and patterns from the BeatPilot social/reel skills (logo, dashboard, world map, heatmap, leaderboard, thumbnail studio references already on file — check current IDs before reusing, they can go stale).

## Programmatic chart requirements

Any chart included in an article or infographic must be built from real BeatPilot data — pull the actual numbers (from Data Analytics, Leaderboard, or a Supabase query if available) rather than illustrating a plausible-looking trend. If the real data isn't accessible in this session, say so and either use a placeholder clearly marked "sample data — replace before publishing" or wait for the data rather than presenting a fabricated chart as real.

## 4:5 cover rules

The 4:5 cover is a distinct asset from the Blog Visual System's standard 16:9 Featured Image — the 4:5 format is for the article's dedicated cover concept (social-first / Pinterest-style crop), not a replacement for the 16:9 hero. Produce both when a full visual plan is requested; don't assume one substitutes for the other. Follow the same Neon Wheel color/typography rules as the Featured Image spec.

## 9:16 explainer rules

For a vertical 9:16 explainer (Reel/Short/TikTok-style), follow the same pipeline already established for BeatPilot's reel content: prefer static image generation (Higgsfield) plus local ffmpeg assembly over full AI video generation, for cost efficiency — see the `reel-video-producer` skill's approach if it's available in this environment. Keep on-screen text in Inter/Inter Display per the type system, and burn in captions rather than relying on platform auto-captions where accuracy matters (e.g. a stat or a feature name).

## Screenshot sourcing

Whenever a real product screenshot is needed, check `BeatPilot Product Information`'s "Current Screenshot Filename" field for that feature first. If it says "Pending — screenshot to be uploaded," don't substitute an illustration and present it as the real feature — tell the user the screenshot doesn't exist yet and ask whether to proceed with a placeholder or wait.

## Approval requirement

Once visuals are produced, save them to the correct `05 Blog Images` subfolder (Featured Images, Open Graph Images, Social Covers, Editorial Artwork, Product Screenshots, Infographics, Charts, Diagrams, or Vertical Videos) and present them to the user for review. **Do not mark the article package as visual-complete or move anything to Publishing Preparation Mode until the human has explicitly approved the visuals.**

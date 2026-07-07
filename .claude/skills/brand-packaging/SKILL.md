---
name: brand-packaging
description: Take one piece of BeatPilot content (image or video) from the "Make Brand Packaging" Drive folder and turn it into a full cross-platform posting package — every ratio/format it's missing gets generated (Higgsfield produces correctly-sized still images, ffmpeg assembles every video), then every resulting placement gets scheduled through Blotato across Instagram, Facebook, TikTok, X, Threads, LinkedIn, Pinterest, and YouTube. Use this whenever asked to run brand packaging, package content for all platforms, turn a post into all formats, or repurpose an asset across social media. Always run this instead of a single manual post whenever the goal is "post this everywhere."
---

# Brand Packaging (BeatPilot)

One input asset in → one full cross-platform package out, scheduled everywhere it can legally/technically go. Higgsfield only ever produces correctly-sized still images; ffmpeg is the only thing that ever builds a video. This skill always **schedules** (never publishes immediately).

## Reference locations

- **Input**: Make Brand Packaging folder — `1NTMeaPaq2SJi0HfKtjUVWo799MHQQQkj`
- **Ratio reference / output staging** — "Social Media Post" folder `1XbXh0AMifIzYjzpILnrxD-ZD_z6L9bFS`, containing 6 ratio buckets:
  | Bucket | Folder ID | Ratio |
  |---|---|---|
  | 9x16 - Vertical Video | `1DjKEiguo4Mf6uquvwPyQihotLh5h06rP` | 9:16 |
  | 1x1 - Square | `1SCTZxi9OJVUsL54MdxgH_WFVW7DWnXD9` | 1:1 |
  | 4x5 - Portrait | `1ss5dJ6OzrPP87aMG_wnHtRs0JuIHwv2_` | 4:5 |
  | 16x9 - Landscape Video | `1W1pO6v-Jhd4D3shGqmhhvZVLricOt36i` | 16:9 |
  | 2x3 - Pinterest Pin | `1nGrzLY66_5ACJ-vxQd2SGzIJMAeH4bnM` | 2:3 |
  | 1.91x1 - Wide Banner | `1Y83zNcfVpqEYgixxztPL_wW_n5qkke_k` | 1.91:1 |
- **Could Not FFmpeg (fallback)**: `1YAMxuyN_e35GYAUC1rTH8kEHAIE9ypXd`
- **Audio Files**: `1yKQcA7A1lix5xJ1Yvc38DXqcQQp47PhW` — pick randomly each time
- **Dedup log**: "Brand-Packaging-Processed-Log" doc in BeatPilot-Info (`1KF05eVO3FVotXQ5oD_KgnSRMTrjwEQ41`) — create if missing. Docs here are create-only (no in-place edit), so append by recreating the doc with prior content + new entry, same pattern as the Ad-Reel Log.

## Environment

This skill needs both Higgsfield network access and local ffmpeg — the cloud Routine sandbox blocks `higgsfield.ai`/`cloudfront.net` egress, so run this via the local Claude Code CLI, not the cloud sandbox (same constraint as `reel-video-producer`). Confirm `which ffmpeg` before starting; if missing, install it. If ffmpeg genuinely can't run this session, don't abort the whole file — fall through to the "Could Not FFmpeg" fallback (below) for just the video-from-still steps and keep going with everything else.

## Standing guardrails (all runs)

- Post to **all** connected platforms — Instagram, Facebook, TikTok, X, Threads, LinkedIn, Pinterest, YouTube. This skill is explicitly exempt from the Instagram-only guardrail that other BeatPilot skills follow.
- **No carousels in this version.** Skip the Carousel row of the content matrix entirely — single-asset placements only.
- **Never publish immediately.** Every `blotato_create_post` call carries an explicit `scheduledTime` — no bare immediate publish, and don't rely on `useNextFreeSlot` alone.
- No fabricated claims, testimonials, or unconfirmed pricing in any generated copy.
- High Voltage brand palette (Voltage Lime `#D1FE17` / Electric Violet `#8A2BE2` / True Black background) and the real logo file carry through every regenerated variant — pass the logo as a reference image to Higgsfield, never let it re-invent the mark.

## Main loop

Run until the Make Brand Packaging folder is empty of unprocessed files or the user stops it manually.

1. **Pick next file.** List Make Brand Packaging contents, cross-check filenames/fileIds against Brand-Packaging-Processed-Log, skip anything already logged.
2. **Classify.** Determine media type (image/video) and native dimensions (ffprobe for video, image header for stills — download locally first).
3. **Match against the 6 ratio buckets.** Compute aspect ratio, match to the closest bucket (allow small tolerance, e.g. 1080x1920 vs 1080x1919). Note which buckets are satisfied and which are gaps.
4. **Fill every gap:**
   - **Gap is an image ratio** → Higgsfield `outpaint_image` to reframe the existing image (or a representative still) into that ratio (don't regenerate from scratch — outpainting preserves the actual creative instead of risking brand/content drift).
   - **Gap is a video ratio** → Higgsfield never generates video. First get a correctly-sized still for that ratio: if the source is already a video, pull a representative frame from it (ffmpeg `-ss ... -vframes 1`) and outpaint that frame to the target ratio; if the source is a still, outpaint it directly to the target ratio. Then ffmpeg builds the actual video from that still — Ken Burns zoom/pan, ~20s, synced to a randomly-picked Audio Files track, H.264/AAC output. Every video in this pipeline, without exception, is assembled by ffmpeg from a Higgsfield-sized still — Higgsfield itself only ever outputs images.
   - **ffmpeg unreachable for a given video-from-still step** → save that ratio's still (already outpainted to the correct aspect ratio) to Could Not FFmpeg instead of skipping it silently, so it's ready for manual video assembly later.
5. **Text post.** Generate one core caption (brand voice, no fabricated claims) plus 3–5 hashtags targeting the demographic (18–34 YouTube beat producers, FL Studio/Ableton, type beat culture). Occasionally (roughly every third asset, or when the log shows the same set repeating) rotate 1–2 hashtags toward a market-gap angle instead of pure demographic targeting — check the log so consecutive posts don't reuse an identical hashtag set. Trim/adapt the caption length per platform limit (X ~280 chars, Threads ~500, others uncapped in practice).
6. **Fan out and schedule.** For every ratio bucket produced (including the ones that were already satisfied natively — nothing skips posting just because no generation was needed), post to every placement below. Call `Blotati:blotato_list_accounts` once per run to resolve real `accountId`/`pageId`/`boardId` values before posting.

## Fan-out table (per ratio bucket → Blotato calls)

| Ratio | Platform | platform value | Notes |
|---|---|---|---|
| 9:16 | Instagram Reel | `instagram` | `mediaType: "reel"` |
| 9:16 | Instagram Story | `instagram` | `mediaType: "story"` |
| 9:16 | Facebook Reel | `facebook` | `mediaType: "reel"`, needs `pageId` |
| 9:16 | Facebook Story | `facebook` | `mediaType: "story"`, needs `pageId` |
| 9:16 | TikTok | `tiktok` | `privacyLevel: "PUBLIC_TO_EVERYONE"` default |
| 9:16 | YouTube Short | `youtube` | needs `title`, `privacyStatus: "public"` |
| 1:1 | Instagram Feed | `instagram` | no `mediaType` |
| 1:1 | Facebook Feed | `facebook` | needs `pageId` |
| 1:1 | LinkedIn Feed | `linkedin` | `pageId` optional (company page) |
| 1:1 | X | `twitter` | |
| 1:1 | Threads | `threads` | |
| 4:5 | Instagram Feed | `instagram` | |
| 4:5 | Facebook Feed | `facebook` | needs `pageId` |
| 4:5 | LinkedIn Feed | `linkedin` | |
| 16:9 | Facebook Video | `facebook` | `mediaType: "video"` (or default), needs `pageId` |
| 16:9 | LinkedIn Video | `linkedin` | |
| 16:9 | X Video | `twitter` | |
| 2:3 | Pinterest | `pinterest` | needs `boardId`, `title` |
| 1.91:1 | Facebook Link Post | `facebook` | `link` field, needs `pageId` |
| 1.91:1 | LinkedIn Link Post | `linkedin` | `link` field |
| 1.91:1 | X Link Preview | `twitter` | `link` field |
| Text (auto-gen) | X | `twitter` | |
| Text (auto-gen) | Threads | `threads` | |
| Text (auto-gen) | Facebook | `facebook` | needs `pageId` |
| Text (auto-gen) | LinkedIn | `linkedin` | |

## Scheduling logic (per placement)

The unit that can't collide is **platform + post-type** (e.g. "Instagram Reel", "Instagram Story", "Facebook Feed") — not just the platform. A single time block can hold an Instagram Reel *and* an Instagram Story together, since those are different types, but it can never hold two Instagram Reels.

1. Call `Blotati:blotato_list_schedules` to see what's already queued, and note it as a set of `(timeBlock, platform, postType)` triples already occupied.
2. For each placement in the fan-out table, walk the standing six-ET-slot daily anchor pattern (same slots social-poster uses) and pick the **first block where that exact platform+type combo is still free** — even if that block already holds other platform/type posts. Roll to the next day if every block that day already has that platform+type taken.
3. Because the fan-out table never repeats a platform+type combo for a single source asset, all placements from one asset can usually land in the *same* time block. The collision case this guards against is a different asset/run already having claimed that platform+type in that block — in which case move to the next block.
4. Call `Blotati:blotato_create_post` with the explicit `scheduledTime` (never immediate, never bare `useNextFreeSlot`).
5. Record each resulting `postSubmissionId` against its placement, block, and platform+type in the log entry for this file.

## ffmpeg pattern (builds every video, any ratio, from a Higgsfield-sized still)

```
ffmpeg -loop 1 -i still.jpg -i audio.mp3 -t 20 \
  -vf "scale=<W>:<H>:force_original_aspect_ratio=increase,crop=<W>:<H>,zoompan=z='min(zoom+0.0006,1.12)':d=1:s=<W>x<H>:fps=30" \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest -movflags +faststart output.mp4
```

Swap `<W>x<H>` for the target ratio's resolution (1080x1920, 1920x1080, etc.). If the audio track runs long or short, trim with `-t` rather than stretching it.

## Logging (Brand-Packaging-Processed-Log)

Per file processed, record: source filename/fileId, date, which buckets were native vs. generated (and via which Higgsfield tool/job id), which video-from-still steps used ffmpeg vs. fell back to Could Not FFmpeg, the hashtag set used, and every placement's Blotato `postSubmissionId` + scheduled time. This is what step 1 of the main loop checks to avoid reprocessing.

## Guardrails

- Never post a carousel — out of scope for this version.
- Never publish immediately; every post must carry an explicit `scheduledTime`.
- Never skip a satisfied-natively bucket's posting step — every ratio bucket the file ends up covering gets posted, not just the generated ones.
- Never invent a feature, testimonial, or price in generated captions.
- Never let a missing ffmpeg session drop a fallback silently — always land the still in Could Not FFmpeg instead.
- Never repeat the exact same hashtag set as the immediately prior logged post.
- Never post a horizontal/16:9 video to YouTube — YouTube only ever receives the 9:16 Short; the 16:9 bucket posts to Facebook Video, LinkedIn Video, and X Video.

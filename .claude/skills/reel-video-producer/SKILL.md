---
name: reel-video-producer
description: Produce vertical (9:16) Instagram Reels for BeatPilot from three sources — Ad References, Producer Tips topics, and Topics - Final carousels — using cheap Higgsfield image generation plus local ffmpeg video assembly instead of expensive Higgsfield video generation, then scheduling each to the next free Blotato slot. Use when asked to run the reel producer, make reels, or turn carousels/ad references into video.
---

# Reel Video Producer (BeatPilot)

Three independent jobs. Each produces one vertical (1080x1920, 9:16) Reel and schedules it via Blotato. The entire cost rationale: Higgsfield only ever generates **still images** here, never video — ffmpeg does the video assembly locally, which is far cheaper than burning video-generation credits for a static-ish 15s clip.

## Shared conventions across all three jobs

**Environment**: confirm ffmpeg is available (`which ffmpeg`) before the first job runs each session; if missing, install it (`apt-get update && apt-get install -y ffmpeg`). If it can't be installed, stop and say so rather than posting a broken or missing video.

**Output spec**: 1080x1920 (9:16), H.264 video, AAC audio, ~15 seconds, `-movflags +faststart`. Add a subtle Ken Burns zoom/pan on static images; where a job stitches multiple images, use short crossfade transitions (0.4-0.6s) rather than hard cuts. Motion reads better than a slide deck, both to viewers and to the algorithm.

**Audio**: pull a file from the Audio Files Drive folder (`1yKQcA7A1lix5xJ1Yvc38DXqcQQp47PhW`). Trim or loop it to match the video's exact duration — don't let it run long or cut off mid-phrase if avoidable.

**Brand**: any newly generated image must use the current "High Voltage" palette (lime + violet) — never the retired blue accent. Never include unverified testimonials, customer claims, or unconfirmed pricing in any on-image text or caption.

**Explain the product, not just the hook**: a reference ad's attention-grabbing structure (a bold headline, a "before/after" split, a vague pain-point line) is worth cloning, but never at the expense of clarity. Every piece of creative this skill produces — ad reels, info-reels, carousel reels — must make it obvious to someone with zero prior context that BeatPilot is a market-intelligence tool for YouTube beatmakers/producers (tracking what's trending, sizing opportunities, analyzing competition) — not just an evocative tagline that assumes the viewer already knows what BeatPilot does. If remapping a reference's headline produces something abstract (e.g. a "chaos to control" hook with no mention of YouTube, beats, producers, or what the tool tracks), rewrite it or add a clear supporting line so the "what is this" question is answered within the first couple seconds, not left for the viewer to guess.

**Posting**: schedule via `blotato_create_post` with `useNextFreeSlot: true`. "All social media" currently means **Instagram only** — that's the only platform connected in Blotato right now, and expansion is blocked pending the open `@beatpilotnow` collision Notion card. Never post to another platform even if one later appears connected, unless a Notion card has explicitly approved it.

## Job 1 — Ad reference → static ad reel

1. Pick an image from the Ad References Drive folder (`1rnNWZBUuWm2k-mDvWjdyRkqHMC4SLimY`). Check the Ad-Reel Log (below) and avoid repeating whichever reference was used last run if there's another option.
2. Use Higgsfield to generate a fresh ad creative based on that reference, remapped to BeatPilot branding, at 1080x1920 (9:16). One image generation call — never generate video directly for this job.
3. ffmpeg: hold the image for 15s with a subtle Ken Burns zoom, synced to an Audio Files track, per the shared output spec.
4. Schedule via Blotato at the next free slot.
5. Append to the **Ad-Reel Log** (a doc in the BeatPilot-Info Drive folder — create if missing): which reference image and which Higgsfield generation were used.

## Job 2 — Producer Tips → info-reel

1. Read the Producer Tips spreadsheet backlog. Check the **Producer-Tips-Reel-Used-Log** (new doc in BeatPilot-Info — create if missing) for topics already used by this job specifically. This is separate from the Used-Topics-Log the carousel pipeline uses — the same topic can support a carousel and a reel without conflict, just not two reels.
2. Generate one static vertical infographic image (1080x1920) for the next unused topic, in the current brand palette.
3. ffmpeg: same static-hold + Ken Burns treatment as Job 1, synced to an Audio Files track.
4. Schedule via Blotato at the next free slot.
5. Mark the topic used in the Producer-Tips-Reel-Used-Log.

## Job 3 — Topics - Final → vertical carousel reel

1. Read the Topics - Final spreadsheet. Check the **Topics-Final-Reel-Used-Log** (new doc in BeatPilot-Info, tracked separately from Job 2's log) for what this job has already used.
2. Generate the next unused topic's carousel as a sequence of vertical (1080x1920) slide images — same content/format as the existing carousel series, rendered to reel dimensions instead of 4:5.
3. ffmpeg: assemble the images in carousel order into one video, each slide holding for an even split of the total runtime, with a short crossfade between each pair, synced to an Audio Files track for the full duration.
4. Schedule via Blotato at the next free slot.
5. Mark the topic used in the Topics-Final-Reel-Used-Log.

## ffmpeg patterns (reference — adapt parameters per job)

Single static image + audio, Ken Burns zoom, 15s:

ffmpeg -loop 1 -i image.jpg -i audio.mp3 -t 15 \
  -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,zoompan=z='min(zoom+0.0008,1.15)':d=1:s=1080x1920:fps=30" \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest -movflags +faststart output.mp4

Multiple images with crossfade (N slides — adapt duration/offsets to slide count and total runtime):

ffmpeg -loop 1 -t <slide_dur> -i img1.jpg -loop 1 -t <slide_dur> -i img2.jpg ... -i audio.mp3 \
  -filter_complex "[0:v]scale=1080:1920,setsar=1[v0];[1:v]scale=1080:1920,setsar=1[v1];...;[v0][v1]xfade=transition=fade:duration=0.5:offset=<slide_dur-0.5>[v01];..." \
  -map "[final]" -map <audio_index>:a -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest -movflags +faststart output.mp4

If audio runs shorter or longer than the assembled video, trim whichever is longer with `-t` rather than stretching either to fit.

## Guardrails

- Never generate video directly through Higgsfield for any of these jobs — image generation + ffmpeg assembly only; that's the entire cost rationale for this skill existing.
- Never post to a platform other than Instagram, regardless of what "all social media" might suggest, until the `@beatpilotnow` collision is resolved.
- Never reuse a topic already marked used in that specific job's own log — the three jobs' logs are independent of each other and of the carousel pipeline's Used-Topics-Log.
- Never fabricate on-image claims, testimonials, or specific prices.
- If ffmpeg isn't available and can't be installed, stop and say so rather than posting a broken or missing video.

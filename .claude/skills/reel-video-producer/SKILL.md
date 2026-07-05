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

**No fabricated claims**: never include unverified testimonials, customer claims, or unconfirmed pricing in any on-image text or caption.

**Logo and URL, every time, no exceptions**: every piece of creative this skill produces must visually include the actual BeatPilot logo graphic — the real file (`Beat-pilot-Logo-1.png`, in the "Beat Pilot Logos" Drive folder, id `14FgLIa5beegGs3jrWNXpZ8zFh2n4ORNf`), never an invented or redrawn version — and the exact URL "www.BeatPilot.io" rendered as on-image text. Pass the logo file as a reference/composite image to Higgsfield on every single generation call so it's actually present and faithfully reproduced in the creative, not described in a text prompt and left to the model to invent. A finished piece missing the real logo graphic, or missing the exact "www.BeatPilot.io" URL text, is incomplete, regardless of how good the rest of the creative is.

**Name YouTube explicitly — don't soften it into generic language**: "AI market intelligence" or "AI-powered analytics" is vague enough to read as a generic business tool and loses BeatPilot's actual differentiator. Every piece of creative must state "YouTube" outright somewhere in the copy (e.g. "YouTube market intelligence for beat producers," not "AI market intelligence for beat producers") — that's the specific mechanism producers actually care about, and dropping it to sound more polished is a regression, not an improvement.

**Optimize for conversion, not template compliance**: the goal of every ad this skill makes is the highest-converting piece of creative it can reason its way to — not a fixed formula applied the same way every time. Treat the structure and rules below as strong defaults to reason from, not a checklist to satisfy mechanically. If a genuinely stronger angle occurs to you for a specific run — a sharper pain callout, a more specific stat, a better pattern interrupt — take it, as long as it doesn't violate the hard guardrails (no fabricated claims/testimonials/pricing, no missing logo/URL, YouTube named explicitly).

*Lead with the pain, not the brand*: the first 1-2 seconds decide whether someone stops scrolling, and that's not where brand recognition does any work yet — the pain callout is what earns the stop. Structure every piece so the pattern-interrupt (the blunt, specific pain statement, in big text) is the very first thing seen, before any logo mark or brand reveal. The logo, the BeatPilot name, and the URL come after the pain has already landed — as the payoff/reveal, not the opener. Never lead with the logo or brand name as the first beat.

*Use real, specific features — never generic trust badges*: "Private and Secure," "Works Anywhere," or similar generic SaaS badges say nothing a producer specifically cares about and read as filler. Use actual named BeatPilot capabilities instead — Niche Score, Upload Timing, Competitor Tracker, Thumbnail Analyzer, Opportunity Engine — since naming a real, specific feature is both more credible and more interesting than a stock trust signal.

*Vary the hook angle run to run*: repeating the same opening line or structure every time causes fatigue and stops working as a scroll-stopper. Rotate between a few distinct angles across runs — a blunt pain callout, a real specific stat framed as a question ("X beats uploaded this week — how many actually got seen?"), a myth-bust ("more uploads ≠ more views") — and check the Ad-Reel Log before generating so this run's angle isn't a repeat of the last one logged.

**Think like a marketer, not just a designer**: before generating any headline, subline, or on-image copy, reason through it the way a marketer writing for a specific audience would — not by cloning a reference's structure and swapping in BeatPilot's name and colors.

*Who you're talking to*: independent YouTube beatmakers and producers, roughly 18-34, mostly US/UK/Canada/Australia. They live in FL Studio or Ableton, post "type beats," check view counts obsessively, and talk in their own shorthand — type beat, tags, niche, drop, upload, algorithm, views. They are not enterprise SaaS buyers. Copy that sounds like generic startup marketing ("streamline your workflow," "unlock your potential," "elevate your process") will read as fake and out of place to this audience. Write the way one producer would talk to another, not the way a landing page talks to a business buyer.

*Their actual pain*: they spend hours making a beat, upload it, and have no real visibility into whether it'll get any traction — no idea what's already oversaturated, what's actually trending, or when to post for the best shot at being seen. Most of their effort goes into beats nobody was searching for in the first place. That's the exact wound BeatPilot addresses: replacing guesswork with real market data before they spend the time producing.

*The throughline every piece of creative should hit, in order*:
1. Name the pain in their language (e.g. "you just spent six hours on a beat nobody's gonna search for")
2. State plainly what BeatPilot is and who it's for — YouTube market intelligence for beat producers — not implied through icons, stated in words
3. Say what it lets them do differently (know what's trending, what's saturated, and when to post, before they make anything)
4. Give a reason to care right now — curiosity or genuine relevance, not a fake urgency countdown

Skip anything resembling "trusted by thousands of producers" or a specific-number social-proof claim — there's no real customer base yet, so that line would be fabricated regardless of how it's phrased. Lean on the specificity of the pain point and the mechanism instead of borrowed credibility.

**Posting**: schedule via `blotato_create_post` with `useNextFreeSlot: true`. "All social media" currently means **Instagram only** — that's the only platform connected in Blotato right now, and expansion is blocked pending the open `@beatpilotnow` collision Notion card. Never post to another platform even if one later appears connected, unless a Notion card has explicitly approved it.

## Job 1 — Ad reference → static ad reel

1. Pick an image from the Ad References Drive folder (`1rnNWZBUuWm2k-mDvWjdyRkqHMC4SLimY`). Check the Ad-Reel Log (below) and avoid repeating whichever reference was used last run if there's another option.
2. Use Higgsfield to generate a fresh ad creative based on that reference, remapped to BeatPilot branding, at 1080x1920 (9:16). One image generation call — never generate video directly for this job.
3. ffmpeg: hold the image for 15s with a subtle Ken Burns zoom, synced to an Audio Files track, per the shared output spec.
4. Schedule via Blotato at the next free slot.
5. Append to the **Ad-Reel Log** (a doc in the BeatPilot-Info Drive folder — create if missing): which reference image, which Higgsfield generation, and which hook angle (pain callout / stat question / myth-bust / other) were used, so the next run can check this and avoid repeating the same angle back to back.

## Job 2 — Producer Tips → info-reel

1. Read the Producer Tips spreadsheet backlog. Check the **Producer-Tips-Reel-Used-Log** (new doc in BeatPilot-Info — create if missing) for topics already used by this job specifically. This is separate from the Used-Topics-Log the carousel pipeline uses — the same topic can support a carousel and a reel without conflict, just not two reels.
2. Generate one static vertical infographic image (1080x1920) for the next unused topic.
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

---
name: could-not-ffmpeg-recovery
description: Clear out BeatPilot's "Could Not FFmpeg" Drive fallback folder — turn every stranded image or video in it into its own finished 30-second, 9:16 (1080x1920) video with an Audio Files track, drop each result into "Make Brand Packaging," then run the brand-packaging skill so those files actually get posted. Use whenever asked to run the could-not-ffmpeg skill, clear out the Could Not FFmpeg folder, recover the ffmpeg fallback folder, process the stranded/leftover ffmpeg files, or catch up anything that got stuck in Could Not FFmpeg — even if the user doesn't say the word "skill." Always run this instead of manually converting one file at a time.
---

# Could Not FFmpeg Recovery (BeatPilot)

The "Could Not FFmpeg" folder is where `brand-packaging` (and other skills) drop stills or clips whenever a video-from-still step couldn't run in that session. This skill is the cleanup pass: it comes back later, when ffmpeg is actually available, and finishes the job — one finished 30-second 9:16 video **per file**, never combined together. Each output lands in "Make Brand Packaging," and this skill then hands off to `brand-packaging` to actually get it posted.

## Reference locations

- **Input**: Could Not FFmpeg folder — `1YAMxuyN_e35GYAUC1rTH8kEHAIE9ypXd`
- **Audio Files**: `1yKQcA7A1lix5xJ1Yvc38DXqcQQp47PhW` — pick randomly per file
- **Output**: Make Brand Packaging folder — `1NTMeaPaq2SJi0HfKtjUVWo799MHQQQkj`
- **Logs folder**: BeatPilot Logs — `1ePuoB4AINakDg-lVtu-uR0kUM5AOZOhM`
  - **Could-Not-FFmpeg-Recovery-Log** (doc, create if missing) — tracks which source files from Could Not FFmpeg have already been converted, so a file already turned into a video is never reprocessed.
  - **Brand-Packaging-Trigger-Log** (doc, create if missing) — tracks each time this skill handed files off to `brand-packaging`, so a hand-off run isn't repeated for files already sent downstream.
  - Docs are create-only (no in-place edit) — append by recreating the doc with prior content + the new entry, same pattern used elsewhere in BeatPilot's skills.

## Environment

This skill only needs local ffmpeg — no Higgsfield calls at all (every video-from-still step here is done directly by ffmpeg, never reframed by Higgsfield first). Confirm `which ffmpeg` before starting; if missing, install it (`apt-get update && apt-get install -y ffmpeg`) or run this via the local Claude Code CLI if the sandbox can't install it. `ffprobe` (ships with ffmpeg) is used to inspect each source file.

## Main loop

Run until the Could Not FFmpeg folder has no unprocessed files left, or the user stops it manually.

1. **List and filter.** List everything in Could Not FFmpeg. Cross-check filenames/fileIds against the Could-Not-FFmpeg-Recovery-Log and skip anything already logged as converted.
2. **Download and classify.** Download the next unprocessed file locally. Run `ffprobe` to determine:
   - Media type: still image vs. video
   - If video: does it already have an audio stream, and what's its duration
3. **Pick an audio track.** Randomly pick one file from Audio Files and download it locally.
4. **Build the 30-second 9:16 video** (see ffmpeg patterns below) — every output is exactly 1080x1920, H.264 video, AAC audio, ~30 seconds, `-movflags +faststart`, regardless of the source's native size or duration.
5. **Upload the result** to Make Brand Packaging with a clear filename (e.g. `recovered_<original-name>.mp4`).
6. **Log it.** Append an entry to Could-Not-FFmpeg-Recovery-Log: source filename/fileId, source type (image/video), audio track used, output filename/fileId, date.
7. **Repeat** for every remaining unprocessed file in the folder.
8. **Hand off.** Once this run's batch is uploaded, check Brand-Packaging-Trigger-Log, log a new entry (date, list of output filenames/fileIds just delivered), then actually run the `brand-packaging` skill so those new files get packaged and scheduled. Don't just log the hand-off and stop — the trigger has to actually fire in the same session, since the user wants this to end in posted content, not just staged files.

## ffmpeg patterns

**Source is a still image** (holds the frame for the full 30s, Ken Burns zoom, audio is the picked track):
```
ffmpeg -loop 1 -i still.jpg -i audio.mp3 -t 30 \
  -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,zoompan=z='min(zoom+0.0004,1.15)':d=1:s=1080x1920:fps=30" \
  -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest -movflags +faststart output.mp4
```

**Source is a video, with its own audio** (crop/pad to 9:16 with plain ffmpeg — no Higgsfield reframe; loop if shorter than 30s, trim if longer; duck the original audio slightly and mix in the picked Audio Files track so the track is always present without burying the source's own sound):
```
ffmpeg -stream_loop -1 -i source_video.mp4 -stream_loop -1 -i audio.mp3 -t 30 \
  -filter_complex "[0:v]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,setsar=1[v];[0:a]volume=1.0[a0];[1:a]volume=0.25[a1];[a0][a1]amix=inputs=2:duration=first:dropout_transition=2[a]" \
  -map "[v]" -map "[a]" -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags +faststart output.mp4
```

**Source is a video with no audio stream** (crop/pad to 9:16, picked track is the sole audio):
```
ffmpeg -stream_loop -1 -i source_video.mp4 -stream_loop -1 -i audio.mp3 -t 30 \
  -filter_complex "[0:v]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,setsar=1[v]" \
  -map "[v]" -map 1:a -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest -movflags +faststart output.mp4
```

`-stream_loop -1` on both inputs is safe even when the source is already longer than 30s — the `-t 30` trim just cuts it at 30s either way, so one pattern covers both the "too short, needs looping" and "already long enough" cases.

## Guardrails

- **One video per source file, never combined.** Each file in Could Not FFmpeg becomes its own 30-second output — never stitch multiple stranded files together into a single video.
- **No Higgsfield calls in this skill.** Reframing/cropping to 9:16 is done entirely with ffmpeg's `scale`+`crop`, per the video-source pattern above — never send a file to Higgsfield to reframe it first.
- **Never reprocess a file already in the Could-Not-FFmpeg-Recovery-Log.** That log is the only source of truth for what's already been converted — always check it before touching a file.
- **Never skip the hand-off.** A run isn't done until `brand-packaging` has actually been triggered on the newly delivered files — staging files in Make Brand Packaging without triggering the hand-off leaves the job half-finished.
- **Never re-trigger brand-packaging for files already logged in Brand-Packaging-Trigger-Log** from a prior run — that log exists specifically so a hand-off isn't repeated for the same batch.
- **Never post directly from this skill.** This skill's job ends at building the video and triggering `brand-packaging` — all actual scheduling/posting logic belongs to `brand-packaging`, not here.
- **Never delete the original file from Could Not FFmpeg.** Leave the source in place; the recovery log is what prevents reprocessing, matching the log-based dedup pattern used elsewhere in BeatPilot's skills.
- **If ffmpeg is missing and can't be installed, stop and say so** rather than leaving a partially-processed batch or silently skipping files.

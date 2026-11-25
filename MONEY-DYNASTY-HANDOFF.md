# MONEY DYNASTY — HANDOFF (for the Money Dynasty Muse account)

You are the Money Dynasty account. This repo is a fully automatic video pipeline.
Read this whole file before you run anything.

## Repo
https://github.com/umair1047666/money-dynasty-auto-video (public)
GitHub account: **umair1047666** (single account for all 5 channel repos).

## What this pipeline does
Takes a script (job) and builds a finished 1080p video automatically:
1. **01 - Clip Search** — finds real stock clips (Pexels → Pixabay → NASA → Wikimedia) + photos (Openverse) + AI stills (Cloudflare FLUX.1) matching each scene.
2. **02 - Voice** — Edge TTS narration.
3. **03 - Render + QC** — normalises clips, concats, renders 1080p with ducked background music + burned-in highlight subtitles, then auto-QC.
4. **04 - Upload** — uploads the finished video to Drive (dry-run by default; real upload only with the "REAL upload" checkbox).

Run the stages **ONE AT A TIME, in order**. Never run two stages at once
(the concurrency group cancels older pending runs).

## Voice — LOCKED
- **Edge TTS ONLY, voice `en-US-GuyNeural`.**
- **NEVER supply your own voice.mp3 / voice.wav.** A provided voice file WINS
  and overrides Edge TTS. Do not put any voice file in a job folder.
- DavisNeural is BROKEN on the Edge endpoint (failed twice, "No audio was
  received"). Do NOT switch back to Davis. GuyNeural is verified working
  (test run md-test-001, voice.mp3 22.2s).
- If Edge TTS ever fails, the pipeline falls back to Piper automatically.
  Tell the user if that happens — do not silently accept Piper.

## Music — LOCKED
- Track: **The_Money_Dynasty.mp3** (Drive file 1NWFvloaB8AgnidPJV_Wo0ectEt9fM0o_,
  ~2:57, measures −11.96 LUFS — LOUD).
- Level: **6% (−24 dB)**, ducked to **~2% (−34 dB)** under the voice.
  (User chose 6% over the recommended 5%.)
- The track is fetched from Drive via secret DRIVE_MUSIC_FILE_ID.
  It is never committed to git. If the track is missing, the render FAILS
  loudly — it never renders silently without music.

## Config (do not change without the user)
- Channel: Money Dynasty (finance, wealth, billionaire documentaries)
- YouTube category: **27 (Education)**
- Visual mix: **40% motion / 60% stills (TARGET only, NOT enforced in config)**
  Per scene the pipeline tries: real clip → real photo + zoom → AI still + zoom
  → title card. So the actual ratio in a video depends on what clips are found,
  not on this target. There is NO AI video generator in the pipeline — "AI
  motion" means AI stills with zoom/pan, not real AI video.
- Final: 1920×1080, 30fps, CRF 18, loudnorm −14 LUFS
- Clips <720p rejected; 720p upscaled to 1080p (lanczos)

## Secrets — 9/9 DONE (Drive-only)
PEXELS, PIXABAY, CLOUDFLARE_API_TOKEN, CLOUDFLARE_ACCOUNT_ID,
DRIVE_REFRESH_TOKEN, DRIVE_CLIENT_ID, DRIVE_CLIENT_SECRET,
DRIVE_FOLDER_ID, DRIVE_MUSIC_FILE_ID.
YouTube secrets were SKIPPED by user decision (Drive-only review).
Do NOT ask the user for secrets. Do NOT copy secrets between repos
(they are write-only; re-enter per repo if ever needed).

## Drive
- Finished videos folder: **"THE MONEY DYNASTY"**
  (ID 1bbRheCJdzLI3fY9YE--2f0YIVpnekUTj)
- Job store (intermediate files): the `pipeline-jobs/` folder inside it.
- Drive DELETE is PERMANENT (not in Trash). Never delete anything.

## How to run a job
1. The user (Umair) gives you a script + scenes. Create the job
   (see `jobs/README.md` or copy `jobs/md-test-001/` as a template).
2. Dispatch **01 - Clip Search** from the Actions tab (branch main).
   Wait for it to succeed. Check the log for the clip count.
3. Dispatch **02 - Voice**. Wait. Check the log says
   "Edge TTS voice generated" (NOT Piper fallback).
4. Dispatch **03 - Render and QC**. Wait. Check QC passed
   (1920x1080, audio present, no black segment over 1s).
5. Dispatch **04 - Upload** (dry-run by default). It verifies the upload
   path. For a REAL upload to the Drive folder, tick the
   "REAL upload (not a dry run)" checkbox when dispatching.
6. Send Umair the Drive link to the finished video. He reviews it.

## Stale-state rule (learned the hard way)
If you ever need to RE-RUN a stage for the same job (e.g. voice failed
and you want to retry it): a re-run does NOTHING ("no job waiting") if
`state.json` still says that stage is done. You MUST reset `state.json`
on BOTH GitHub (`jobs/<job>/state.json`) AND the Drive job store
(set that stage `ok: false`, clear `voice` / `voice_duration_s`,
set `stage` back to the previous stage) AND delete the old voice file
from the Drive job store — BEFORE re-dispatching. Otherwise the re-run
finishes in seconds having done nothing.

## Browser log rule
GitHub's log view can FREEZE on an old line while the job actually
finishes. Never trust a frozen step log. Always check the run's OVERALL
status badge and total duration first.

## GATE — read this before scaling
First make + verify **ONE full 20-minute video** before scaling to
10/week. A short test does NOT prove music looping, subtitles, or a
large render. A 20-min video takes ~30–50 min total pipeline time,
unattended. 10/week = 10 scripts/jobs — a weekly target, not daily.

## Rules
- **NEVER upload to YouTube without Umair's explicit OK.**
- **NEVER delete anything** (no videos, no files, no Drive files).
- **NEVER ask for API keys, tokens, or passwords in chat.** Secrets are
  already set. If one is missing, tell Umair which one and let him set it.
- One Muse account per channel at a time. You are the Money Dynasty
  account. Do NOT touch Steel Atlas, Cosmos Rising, Umbral Hour, or
  Ages of Ruin repos or their metadata.
- If a stage fails, read the FULL log, report the exact error line to
  Umair, and do NOT guess the cause twice.
- Pexels: 200 requests/hr + 20,000/mo per key. Cloudflare free:
  10,000 Neurons/day (~57–173 AI stills/day — plan on 57).
  Money Dynasty has its OWN separate Cloudflare account (own 10,000/day).

## Verified test (md-test-001, Vanguard $13T, 2026-11-25)
- Stage 01 PASSED: 3 clips, all Pexels 4K.
- Stage 02 PASSED: Edge TTS GuyNeural, voice.mp3 22.2s.
- Stage 03 PASSED: final.mp4 22.20s, 1920x1080, QC passed, music mixed.
- Stage 04: upload dry-run.
- Final test video on Drive: file 1aT2S9JSoPoDe3NBxhfSc3sRxblp0xvgX
- User APPROVED the test video (Guy voice + music) on 2026-11-25.

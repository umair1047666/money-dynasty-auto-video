# Jobs — how to create a video job

A job is one folder under `jobs/`, named `md-XXXX` (e.g. `md-0001`).
Each job folder contains one file: `job.json`.

## job.json format

```json
{
  "job_id": "md-0001",
  "title": "How Vanguard Became a $13 Trillion Giant",
  "description": "One-line summary of the video.",
  "status": "queued",
  "tags": ["vanguard", "index funds", "money"],
  "voice_file": null,
  "music_track": "track.mp3",
  "script": "Full narration text, all scenes joined. [HL]highlight phrases[/HL] mark the key-text subtitles.",
  "scenes": [
    {
      "id": 1,
      "duration_s": 6,
      "narration": "Vanguard manages thirteen trillion dollars. More than the economy of most countries.",
      "keywords": ["stock exchange", "wall street", "trading floor"],
      "fallback_prompt": "dramatic photograph of the New York Stock Exchange building on Wall Street"
    }
  ]
}
```

## Field rules

- **job_id**: `md-` + number, unique per job. Never reuse an id.
- **title**: the YouTube title (without "TEST —" unless it is a test).
- **status**: always `"queued"` for a new job.
- **voice_file**: always `null`. Edge TTS generates the voice. NEVER supply a
  voice file — a provided file wins and overrides Edge TTS.
- **music_track**: always `"track.mp3"`.
- **script**: the full narration, all scenes joined in order. Must match the
  scene narrations exactly. Mark key phrases with `[HL]...[/HL]` — only those
  phrases appear as on-screen subtitles (key-text only, ~5% of narration).
  Do NOT mark whole sentences; mark short key phrases only.
- **scenes**: one per visual. Order matters (scene 1 first).
  - **id**: 1, 2, 3, ... in order.
  - **duration_s**: how long this scene's visual lasts (seconds).
    Scene lengths: hook 2–3s, body 3–5s, evidence 6–10s, peaks 1.5–3s.
    The narration for the scene must FIT its duration: Edge TTS speaks
    ~13–15 characters per second (~2.5 words/sec). So a 6s scene needs
    ~80–90 characters of narration; a 10s scene ~130–150 characters.
    Keep the total of all duration_s close to the voice length
    (a 20-min video = ~1200s total, ~150–160 scenes at ~7–8s each).
  - **narration**: this scene's spoken text. Keep it sized to duration_s
    (see above). It must appear verbatim inside `script`.
  - **keywords**: 2–4 search terms for the real clip (Pexels/Pixabay/NASA/
    Wikimedia). Specific beats generic: "vanguard headquarters building"
    beats "money". These are also used for the relevance gate.
  - **fallback_prompt**: a photo-style prompt for the AI still, used only
    if no real clip and no real photo is found. Describe a photograph,
    not a video: "photograph of ...". Never ask for text, logos, or
    readable words in the image.

## What the pipeline does per scene (no ratio to set)

There is NO motion/still ratio setting. For each scene the pipeline tries,
in order: real video clip → real photo + zoom → AI still + zoom → title card.
So the actual motion/still mix in a video depends on which clips are found.
The 40% motion / 60% stills figure is a TARGET, not an enforced setting.
There is no AI video generator — "AI motion" means AI stills with zoom/pan.

## Voice and music (locked, do not change)

- Voice: Edge TTS `en-US-GuyNeural` only. DavisNeural is broken on the
  Edge endpoint. Never supply a voice file.
- Music: The_Money_Dynasty.mp3 at 6% (−24 dB), ducked to ~2% (−34 dB).
- Final video: 1920×1080 landscape, 30fps, CRF 18, loudnorm −14 LUFS.
- YouTube category: 27 (Education).

## How to submit a job

The Money Dynasty account writes the job.json and gives it to Umair.
Umair forwards it to the pipeline account, which pushes it to
`jobs/<job_id>/job.json` on GitHub (branch main). Then the 4 stages are
dispatched one at a time from the Actions tab:
01 - Clip Search → 02 - Voice → 03 - Render and QC → 04 - Upload (dry-run
by default; tick "REAL upload" only for a real upload, and only after
Umair approves the finished video).

## Template

Copy `jobs/md-test-001/job.json` as the starting template. It is the
verified pilot: 3 scenes (6/8/10s), Vanguard $13T, GuyNeural voice 22.2s,
final.mp4 22.20s 1920×1080, QC passed, upload dry-run passed.

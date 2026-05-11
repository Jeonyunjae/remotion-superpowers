---
name: transcribe
description: Transcribe audio or video files to text with word-level timestamps. Uses your configured provider (faster-whisper, Remotion Whisper, or KIE cloud) for accurate transcription, outputs SRT subtitles or JSON captions for use with Remotion's captions system.
---

# Transcribe — Audio/Video to Text

You are helping the user transcribe audio or video files into text with precise timestamps.

**Load the `remotion-production` skill** for the `captions-workflow` and `model-providers` rules.

## Workflow

### 1. Identify the Source File

Find audio/video files to transcribe:

```bash
# Check common locations
ls public/audio/ public/footage/ public/ 2>/dev/null | grep -E '\.(mp3|wav|m4a|ogg|mp4|mov|webm)$'
```

Ask the user which file to transcribe if multiple exist.

### 2. Check Provider Configuration

Read `config.yaml` to determine the subtitle/transcription provider:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, ask the user: "Would you like to run `/select-models` to choose your transcription provider, or use the default (faster-whisper, free)?"

### 3. Transcribe

Use the configured provider.

#### faster-whisper provider (FREE):

```bash
python3 << 'PYEOF'
from faster_whisper import WhisperModel
import os

# Model from config.yaml (default: large-v3-turbo)
# Device from config.yaml (default: cpu)
model = WhisperModel("large-v3-turbo", device="cpu", compute_type="int8")
# For GPU: device="cuda", compute_type="float16"

audio_path = "[path to audio/video file]"
segments, info = model.transcribe(audio_path, word_timestamps=True)

print(f"Language: {info.language} (probability: {info.language_probability:.2f})")

os.makedirs("public/subtitles", exist_ok=True)

# Generate SRT file
all_segments = list(segments)
with open("public/subtitles/captions.srt", "w", encoding="utf-8") as f:
    for i, segment in enumerate(all_segments, 1):
        start = segment.start
        end = segment.end
        sh, sm, ss = int(start//3600), int((start%3600)//60), int(start%60)
        sms = int((start % 1) * 1000)
        eh, em, es = int(end//3600), int((end%3600)//60), int(end%60)
        ems = int((end % 1) * 1000)
        
        f.write(f"{i}\n")
        f.write(f"{sh:02d}:{sm:02d}:{ss:02d},{sms:03d} --> {eh:02d}:{em:02d}:{es:02d},{ems:03d}\n")
        f.write(f"{segment.text.strip()}\n\n")

# Also generate word-level JSON for animated captions
import json
words = []
for segment in all_segments:
    if segment.words:
        for word in segment.words:
            words.append({
                "text": word.word,
                "startMs": int(word.start * 1000),
                "endMs": int(word.end * 1000),
                "confidence": round(word.probability, 3) if word.probability else 0.9
            })

with open("public/subtitles/captions.json", "w", encoding="utf-8") as f:
    json.dump(words, f, ensure_ascii=False, indent=2)

print(f"SRT saved to public/subtitles/captions.srt")
print(f"JSON saved to public/subtitles/captions.json")
print(f"Segments: {len(all_segments)}")
PYEOF
```

**Model options:** tiny, base, small, medium, large-v3, large-v3-turbo (recommended).

**Note:** First run downloads the model weights. `large-v3-turbo` is ~809MB but provides the best quality-to-speed ratio. For faster results on CPU, use `small` or `base`.

#### whisper-local provider (FREE):

Remotion's built-in Whisper.cpp integration:

```bash
# Install Whisper.cpp locally (one-time)
npx remotion add @remotion/install-whisper-cpp
```

```typescript
import { installWhisperCpp, transcribe } from "@remotion/install-whisper-cpp";

// One-time install
await installWhisperCpp({ version: "1.5.5" });

// Transcribe
const result = await transcribe({
  inputPath: "[path to audio/video file]",
  whisperPath: ".whisper",
  model: "medium",  // tiny, base, small, medium, large
  tokenLevelTimestamps: true,
});
```

This runs entirely on-device — no API key needed. Native integration with `@remotion/captions`.

#### kie provider (PAID):

```
Use remotion-media generate_subtitles:
- input: [path to audio/video file]
- project_path: [project root path]
```

This generates an SRT file with timestamps saved to the project.

### 4. Present the Transcript

Show the transcription with timestamps:

```
Transcription: [filename]

[00:00:00 --> 00:00:03] Welcome to our product demo
[00:00:03 --> 00:00:07] Today we're going to show you how easy it is
[00:00:07 --> 00:00:11] to create professional videos with code
...

Total duration: [X]s
Word count: [X] words
```

### 5. Output Formats

Ask what format the user needs:

**SRT (default)** — Standard subtitle format:
```srt
1
00:00:00,000 --> 00:00:03,200
Welcome to our product demo

2
00:00:03,200 --> 00:00:07,100
Today we're going to show you how easy it is
```

**JSON Captions** — For Remotion's `@remotion/captions`:
```json
[
  { "text": " Welcome", "startMs": 0, "endMs": 800, "confidence": 0.95 },
  { "text": " to", "startMs": 800, "endMs": 1000, "confidence": 0.98 },
  { "text": " our", "startMs": 1000, "endMs": 1200, "confidence": 0.97 }
]
```

**Plain text** — Just the words, no timestamps:
```
Welcome to our product demo. Today we're going to show you how easy it is to create professional videos with code.
```

### 6. Common Use Cases

After transcription, suggest next steps:

- **Add captions** — Run `/add-captions` to create TikTok-style animated subtitles
- **Sync animations** — Use timestamps to trigger visual elements at specific words
- **Create a script** — Edit the transcript as the basis for a new voiceover
- **Translate** — Use the transcript to create versions in other languages
- **Content repurpose** — Extract key quotes for social media clips

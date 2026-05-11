---
name: add-music
description: Generate background music and add it to your Remotion project. Describe the mood/genre you want, and this will generate a track and wire it into your composition with proper fade in/out.
---

# Add Music — Generate & Integrate Background Music

You are helping the user add background music to their Remotion project.

**Load the `remotion-production` skill** for the `music-scoring`, `audio-integration`, and `model-providers` rules.

## Workflow

### 1. Understand the Mood

If not already described, ask the user:
- **Genre** — Electronic, jazz, classical, lo-fi, ambient, rock, orchestral, pop?
- **Mood** — Upbeat, calm, dramatic, inspiring, playful, cinematic?
- **Tempo** — Fast, medium, slow?
- **Vocals** — Instrumental only (recommended for background), or with vocals?

### 2. Check Provider Configuration

Read `config.yaml` to determine the music provider:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, ask the user: "Would you like to run `/select-models` to choose your music provider, or use the default (musicgen, free)?"

### 3. Craft the Prompt

Write a descriptive prompt. Good prompts include:
- Genre and style
- Mood and energy level
- Instrumentation
- "no vocals" (unless vocals are wanted)
- Tempo or BPM reference

**Good prompt examples:**
- "upbeat electronic background music, energetic, no vocals, modern tech startup vibe"
- "calm acoustic guitar, warm and friendly, no vocals, coffee shop atmosphere"
- "epic cinematic orchestral, building tension and release, no vocals, trailer style"
- "chill lo-fi hip hop beats, relaxed and dreamy, no vocals, study music vibe"

### 4. Generate Music

Use the configured provider to generate the track.

#### musicgen provider (FREE):

```bash
python3 << 'PYEOF'
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import scipy.io.wavfile
import numpy as np

model_name = "facebook/musicgen-small"  # or musicgen-medium, musicgen-large
processor = AutoProcessor.from_pretrained(model_name)
model = MusicgenForConditionalGeneration.from_pretrained(model_name)

inputs = processor(
    text=["[music prompt]"],
    padding=True,
    return_tensors="pt"
)

# Duration control: 256 tokens = ~5s, 512 = ~10s, 1024 = ~20s, 1536 = ~30s
audio_values = model.generate(**inputs, max_new_tokens=1536)
audio_data = audio_values[0, 0].cpu().numpy()

sampling_rate = model.config.audio_encoder.sampling_rate
scipy.io.wavfile.write("public/audio/music.wav", sampling_rate, audio_data)
print(f"Music saved to public/audio/music.wav")
print(f"Duration: {len(audio_data)/sampling_rate:.1f}s")
PYEOF
```

**Note:** First run downloads the model weights (~300MB for small, ~1.5GB for medium). MusicGen generates instrumental music only — "no vocals" is the default behavior.

#### ace-step provider (FREE, requires local server):

```bash
# Requires ACE-Step running on localhost:8001
curl -X POST http://localhost:8001/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[music prompt]",
    "duration": 30,
    "seed": -1
  }' \
  -o public/audio/music.wav

echo "Music saved to public/audio/music.wav"
```

If the ACE-Step server is not running, tell the user to start it:
```bash
cd /path/to/ACE-Step && python server.py --port 8001
```

#### suno (KIE) provider (PAID):

```
Use remotion-media generate_music:
- prompt: "[music prompt, always include 'no vocals' if instrumental]"
- project_path: [project root path]
```

### 5. Wire into Composition

Add with fade in/out:

```tsx
import { Audio, interpolate, staticFile, useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
const { durationInFrames, fps } = useVideoConfig();

const musicVolume = interpolate(
  frame,
  [0, fps, durationInFrames - 2 * fps, durationInFrames],
  [0, 0.25, 0.25, 0],
  { extrapolateLeft: "clamp", extrapolateRight: "clamp" }
);

<Audio src={staticFile("[returned-file-path]")} volume={musicVolume} />
```

### 6. Adjust for Voiceover (if present)

If the project has a voiceover, implement volume ducking — lower music to 0.1 during narration, 0.3 during silent parts. See `audio-integration` rule for the pattern.

### 7. Preview

Tell the user to check the audio in the Remotion preview (`npm run dev`).

Offer adjustments:
- Volume too loud/quiet?
- Wrong mood? Regenerate with a different prompt.
- Music too short? Use `<Loop>` to repeat it.
- Need different music for different sections? Generate multiple tracks.

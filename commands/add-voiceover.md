---
name: add-voiceover
description: Generate a voiceover narration and add it to your Remotion project. Provide a script or describe what the narration should say, and this will generate TTS audio and wire it into your composition.
---

# Add Voiceover — Generate & Integrate Narration

You are helping the user add a voiceover narration to their Remotion project.

**Load the `remotion-production` skill** for the `voiceover-sync`, `audio-integration`, and `model-providers` rules.

## Workflow

### 1. Get the Script

If the user hasn't provided a script:
- Ask what the video is about
- Ask about tone (professional, casual, energetic, calm)
- Write the narration script
- Present for approval

### 2. Check Provider Configuration

Read `config.yaml` to determine the TTS provider:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, ask the user: "Would you like to run `/select-models` to choose your TTS provider, or use the default (edge-tts, free)?"

### 3. Choose Voice

Voice options depend on the configured provider.

#### If provider is `edge-tts`:

**Korean voices:**
- **ko-KR-SunHiNeural** — Clear, professional female (default for Korean)
- **ko-KR-InJoonNeural** — Warm, natural male
- **ko-KR-HyunsuNeural** — Friendly, conversational male
- **ko-KR-JiMinNeural** — Bright, energetic female
- **ko-KR-SeoHyeonNeural** — Calm, soothing female

**English voices:**
- **en-US-AriaNeural** — Energetic female (default for English)
- **en-US-GuyNeural** — Professional, warm male
- **en-US-JennyNeural** — Friendly, conversational female
- **en-US-DavisNeural** — Casual, youthful male
- **en-GB-SoniaNeural** — British, clear female
- **en-GB-RyanNeural** — British, professional male

To list all available voices:
```bash
edge-tts --list-voices | grep -E "^(ko-KR|en-US|en-GB|ja-JP)"
```

#### If provider is `kokoro`:

- **af_heart** — Warm, natural female (default)
- **af_bella** — Clear, professional female
- **am_adam** — Professional male
- **am_michael** — Warm male

Note: Kokoro supports English only. For Korean, suggest switching to edge-tts.

#### If provider is `coqui-xtts`:

Ask the user for a reference audio file (6-30 seconds of clear speech). The voice will be cloned from this sample. Supports Korean and English.

#### If provider is `elevenlabs` (KIE) or `elevenlabs-direct`:

- **Eric** — Warm, professional male (default)
- **Rachel** — Clear, engaging female
- **Aria** — Energetic female
- **Roger** — Deep, authoritative male
- **Sarah** — Friendly, conversational female
- **Laura** — Calm, soothing female
- **Charlie** — Casual, youthful male

For voice cloning or custom voices, use the dedicated ElevenLabs MCP (requires `ELEVENLABS_API_KEY`).

### 4. Generate TTS

Use the configured provider to generate the voiceover.

#### edge-tts provider:

```bash
# Generate voiceover
edge-tts --text "[approved script]" --voice "[chosen voice ID]" --write-media public/audio/voiceover.mp3

# Get duration for composition timing
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.mp3
```

**Tip:** For long scripts, edge-tts handles them in a single call. No need to split.

#### kokoro provider:

```bash
python3 -c "
from kokoro import KPipeline
import soundfile as sf

pipeline = KPipeline(lang_code='a')
audio, sr = pipeline('[approved script]', voice='[chosen voice]')
sf.write('public/audio/voiceover.wav', audio, sr)
print(f'Duration: {len(audio)/sr:.2f}s')
"
```

#### coqui-xtts provider:

```bash
python3 -c "
from TTS.api import TTS

tts = TTS('tts_models/multilingual/multi-dataset/xtts_v2')
tts.tts_to_file(
    text='[approved script]',
    speaker_wav='[path/to/reference_audio.wav]',
    language='ko',  # or 'en' for English
    file_path='public/audio/voiceover.wav'
)
"
```

Get duration:
```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.wav
```

#### elevenlabs (KIE) provider:

```
Use remotion-media generate_tts:
- text: [the approved script]
- voice: [chosen voice]
- project_path: [project root path]
```

Record the returned **file path** and **duration**.

#### elevenlabs-direct provider:

```
Use ElevenLabs MCP text_to_speech:
- text: [the approved script]
- voice_id: [chosen voice]
```

Get the duration:
```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.mp3
```

### 5. Wire into Composition

Add the voiceover `<Audio>` component:

```tsx
import { Audio, staticFile } from "remotion";

// In your composition:
<Audio src={staticFile("[returned-file-path]")} volume={1} />
```

### 6. Adjust Composition Duration

If the voiceover is longer than the current composition:

```tsx
// Update durationInFrames in the Composition registration
durationInFrames: Math.ceil([voiceover_duration_seconds] * fps) + 60 // +2s padding
```

### 7. Generate Captions (optional)

Offer to generate subtitles. Use the subtitle provider from `config.yaml`:

#### faster-whisper provider:

```bash
python3 -c "
from faster_whisper import WhisperModel
model = WhisperModel('large-v3-turbo', device='cpu', compute_type='int8')
segments, info = model.transcribe('public/audio/voiceover.mp3', word_timestamps=True)
with open('public/subtitles/captions.srt', 'w', encoding='utf-8') as f:
    for i, seg in enumerate(segments, 1):
        start = seg.start
        end = seg.end
        sh, sm, ss, sms = int(start//3600), int((start%3600)//60), int(start%60), int((start%1)*1000)
        eh, em, es, ems = int(end//3600), int((end%3600)//60), int(end%60), int((end%1)*1000)
        f.write(f'{i}\n{sh:02d}:{sm:02d}:{ss:02d},{sms:03d} --> {eh:02d}:{em:02d}:{es:02d},{ems:03d}\n{seg.text.strip()}\n\n')
print('SRT saved to public/subtitles/captions.srt')
"
```

#### KIE provider:

```
Use remotion-media generate_subtitles:
- input: [voiceover file path]
- project_path: [project root path]
```

### 8. Duck Background Music (if present)

If the project already has background music, lower its volume during voiceover. See `audio-integration` rule for the ducking pattern.

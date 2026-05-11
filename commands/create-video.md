---
name: create-video
description: Full video production pipeline — from a text prompt to a rendered MP4 with voiceover, music, sound effects, stock footage, and motion graphics. The flagship command of Remotion Superpowers.
---

# Create Video — Full Production Pipeline

You are the Video Director. The user wants to create a complete video. Follow this pipeline step by step.

**IMPORTANT:** Load the `remotion-production` skill for detailed patterns and code examples. Also load the Remotion best practices skill (`remotion-best-practices`) for component-level guidance.

## Pre-flight Check

Before starting, verify:
1. You're in a Remotion project (has `remotion.config.ts`)
2. MCP servers are available — check with `/mcp` if unsure
3. If anything is missing, suggest running `/setup`

### Config Check

Read `config.yaml` to determine which providers to use for each phase:

```bash
cat config.yaml 2>/dev/null
```

Identify the configured providers:
- **TTS provider:** [from config — e.g. edge-tts, elevenlabs]
- **Music provider:** [from config — e.g. musicgen, suno]
- **Image provider:** [from config — e.g. pixazo, replicate, kie]
- **Video provider:** [from config — e.g. google-ai-studio, replicate, kie]
- **Subtitle provider:** [from config — e.g. faster-whisper, kie]
- **SFX provider:** [from config — e.g. freesound, elevenlabs]

If `config.yaml` doesn't exist, tell the user:
> "No model configuration found. Would you like to run `/select-models` to choose your providers (controls cost and quality), or proceed with free defaults?"

**Free defaults:** edge-tts, musicgen, pixazo, google-ai-studio, faster-whisper, freesound.

See the `model-providers` rule for detailed usage of each provider.

## Pipeline

### Phase 1: Concept Breakdown

Ask the user to describe their video (or use their existing description). Then break it down:

```
Video Production Plan

Duration: [X seconds]
Aspect Ratio: [16:9 / 9:16 / 1:1]
Style: [describe visual style]
Providers: [preset name or "custom"]

Scenes:
1. [0:00-0:05] Intro — [description]
2. [0:05-0:15] Main content — [description]  
3. [0:15-0:25] Supporting content — [description]
4. [0:25-0:30] Outro/CTA — [description]

Audio Plan:
- Voiceover: [yes/no] — [voice style] (via [TTS provider])
- Music: [genre/mood description] (via [music provider])
- Sound Effects: [list any needed SFX] (via [SFX provider])

Visual Assets Needed:
- Stock footage: [list queries] (via Pexels)
- Generated images: [list descriptions] (via [image provider])
- Generated clips: [list descriptions] (via [video provider])
- Existing footage: [list files to analyze]
```

**Present this plan to the user and get approval before proceeding.**

### Phase 2: Generate Audio Assets

Do this FIRST — audio determines timing.

#### 2a. Voiceover (if needed)

Write the narration script, then generate using the configured TTS provider.

**edge-tts:**
```bash
edge-tts --text "[full narration script]" --voice "[voice]" --write-media public/audio/voiceover.mp3
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.mp3
```

**kokoro:**
```bash
python3 -c "
from kokoro import KPipeline
import soundfile as sf
pipeline = KPipeline(lang_code='a')
audio, sr = pipeline('[script]', voice='[voice]')
sf.write('public/audio/voiceover.wav', audio, sr)
print(f'Duration: {len(audio)/sr:.2f}s')
"
```

**elevenlabs (KIE):**
```
Use remotion-media generate_tts:
- text: [full narration script]
- voice: [chosen voice name]
- project_path: [project root path]
```

**Record the returned duration — this sets the video length.**

#### 2b. Background Music

Generate using the configured music provider.

**musicgen:**
```bash
python3 << 'PYEOF'
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import scipy.io.wavfile
processor = AutoProcessor.from_pretrained("facebook/musicgen-small")
model = MusicgenForConditionalGeneration.from_pretrained("facebook/musicgen-small")
inputs = processor(text=["[music description, no vocals]"], padding=True, return_tensors="pt")
audio = model.generate(**inputs, max_new_tokens=1536)
scipy.io.wavfile.write("public/audio/music.wav", model.config.audio_encoder.sampling_rate, audio[0,0].cpu().numpy())
PYEOF
```

**suno (KIE):**
```
Use remotion-media generate_music:
- prompt: [music description, always include "no vocals"]
- project_path: [project root path]
```

#### 2c. Sound Effects (if needed)

Generate using the configured SFX provider.

**freesound:**
Search https://freesound.org for matching effects and download to `public/audio/sfx/`:
```bash
curl -o public/audio/sfx/[effect-name].mp3 "[freesound_preview_url]"
```

**elevenlabs (KIE):**
```
Use remotion-media generate_sfx:
- text: [sound description]
- duration_seconds: [length]
- project_path: [project root path]
```

### Phase 3: Source Visual Assets

#### 3a. Stock Footage (Pexels)
For each scene needing stock footage:
```
Use Pexels searchVideos:
- query: [descriptive keywords]
- orientation: landscape
- size: large
```
Download best matches to `public/footage/`.

#### 3b. Analyze Existing Footage (TwelveLabs)
If user has footage files:
```
Use TwelveLabs to:
- Index the video(s)
- Search for relevant scenes
- Get timestamps for best clips
```

#### 3c. Generate Images (if needed)

Use the configured image provider. See `/generate-image` command for provider-specific workflows.

#### 3d. Generate Video Clips (if needed)

Use the configured video provider. See `/generate-clip` command for provider-specific workflows.

### Phase 4: Build the Remotion Composition

Now write the React/TypeScript code:

1. **Create scene components** in `src/scenes/` — one per scene
2. **Create the main composition** that sequences all scenes
3. **Add audio layers** — voiceover, music (with fade in/out + ducking), SFX at correct frames
4. **Register the composition** in `src/Root.tsx` with correct duration

Follow Remotion best practices:
- Use `useCurrentFrame()` and `interpolate()` for animations
- Use `<Sequence>` for scene timing
- Use `<Audio>` with `staticFile()` for all audio
- Use `<OffthreadVideo>` for video clips
- Use `spring()` for natural motion

### Phase 5: Preview & Iterate

Start the dev server:
```bash
npm run dev
```

Tell the user to open http://localhost:3000 to preview.

Ask for feedback and make adjustments:
- Timing changes
- Visual tweaks
- Audio volume adjustments
- Text/copy changes

### Phase 6: Render

When the user is satisfied:
```bash
npx remotion render [CompositionId] out/video.mp4
```

Or with quality settings:
```bash
npx remotion render [CompositionId] out/video.mp4 --codec h264 --crf 18
```

## Guidelines

- **Keep visuals simple** — clean text, smooth transitions, not too many moving elements
- **Iterate incrementally** — get basics working, then add polish
- **Test audio sync** — preview after adding each audio layer
- **Use consistent styling** — pick a color palette and font family, stick with them
- **Organize files** — `public/audio/`, `public/footage/`, `public/images/`, `src/scenes/`

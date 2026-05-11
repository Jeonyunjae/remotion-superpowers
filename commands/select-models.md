---
name: select-models
description: Interactively choose which AI model provider to use for each function (TTS, music, image, video, subtitles, SFX). Switch between free local models and paid cloud APIs to control costs.
---

# Select Models — Provider Configuration

You are helping the user choose which AI providers to use for their Remotion Superpowers setup. This controls cost, quality, and speed tradeoffs for every media generation function.

**Load the `remotion-production` skill** for the `model-providers` rule — it has detailed setup instructions for every provider.

## Workflow

### 1. Read Current Configuration

```bash
cat config.yaml 2>/dev/null || echo "No config.yaml found"
```

If `config.yaml` doesn't exist, you will create one. If it exists, show the current settings.

### 2. Present Preset Options

Ask the user to choose a preset or go custom:

```
Which configuration preset would you like?

1. FREE — All local/free providers, no API costs
   TTS: edge-tts (Microsoft neural voices, Korean support)
   Music: MusicGen (Meta, runs locally)
   Image: Pixazo (free cloud API)
   Video: Google AI Studio (free Veo, rate limited)
   Subtitle: faster-whisper (local Whisper)
   SFX: Freesound (free library)
   Cost: $0 | Requires: Python, ~2GB disk for models
   pip install: edge-tts faster-whisper transformers scipy soundfile

2. BUDGET — Free where possible, paid for video/image quality
   TTS: edge-tts (free)
   Music: MusicGen (free, local)
   Image: Replicate FLUX (pay-per-use, ~$0.003/image)
   Video: Replicate Wan 2.5 (pay-per-use, ~$0.05/clip)
   Subtitle: faster-whisper (free, local)
   SFX: Freesound (free)
   Cost: ~$0.05-0.50/video | Requires: REPLICATE_API_TOKEN

3. PREMIUM — Best quality, all KIE cloud services
   TTS: ElevenLabs via KIE (premium voices)
   Music: Suno via KIE (professional quality)
   Image: KIE (fast, integrated)
   Video: Veo 3.1 via KIE (highest quality)
   Subtitle: KIE Whisper (cloud, fast)
   SFX: ElevenLabs via KIE (custom generation)
   Cost: Depends on KIE plan | Requires: KIE_API_KEY

4. CUSTOM — Choose each provider individually
```

### 3. Custom Configuration (if chosen)

Walk through each function one by one:

#### 3a. TTS (Text-to-Speech)

```
TTS Provider:

a) edge-tts [FREE] — Microsoft neural voices via edge-tts Python package
   Korean: ko-KR-SunHiNeural (female), ko-KR-InJoonNeural (male)
   English: en-US-AriaNeural, en-US-GuyNeural, en-GB-SoniaNeural, +300 voices
   Quality: Very good | Speed: Fast | Install: pip install edge-tts

b) kokoro [FREE] — Lightweight 82M parameter TTS model
   Voices: af_heart, af_bella, am_adam, am_michael
   Quality: Good | Speed: Fast | Install: pip install kokoro soundfile
   Note: English only, no Korean support

c) coqui-xtts [FREE] — Voice cloning TTS
   Voices: Clone any voice from a reference audio file
   Quality: Good | Speed: Medium | Install: pip install TTS
   Note: Supports Korean, requires reference audio for cloning

d) elevenlabs (KIE) [PAID] — ElevenLabs via KIE API
   Voices: Eric, Rachel, Aria, Roger, Sarah, Laura, Charlie
   Quality: Excellent | Speed: Fast | Requires: KIE_API_KEY

e) elevenlabs-direct [PAID] — Direct ElevenLabs API
   Voices: All ElevenLabs voices + voice cloning
   Quality: Excellent | Speed: Fast | Requires: ELEVENLABS_API_KEY
```

#### 3b. Music Generation

```
Music Provider:

a) musicgen [FREE] — Meta's MusicGen model, runs locally
   Models: musicgen-small (300M), musicgen-medium (1.5B), musicgen-large (3.3B)
   Quality: Good | Speed: Slow (30s-2min) | Install: pip install transformers scipy
   Note: Best with GPU, works on CPU (slower)

b) ace-step [FREE] — ACE-Step local music generation
   Quality: Very good | Speed: Fast with GPU
   Requires: Local REST server on localhost:8001
   Install: See model-providers rule for setup

c) suno (KIE) [PAID] — Suno via KIE API
   Quality: Excellent (professional grade) | Speed: Fast
   Requires: KIE_API_KEY
```

#### 3c. Image Generation

```
Image Provider:

a) pixazo [FREE] — Pixazo cloud API
   Models: flux-schnell (fast), flux-dev (quality)
   Quality: Good | Speed: Fast | No API key needed for basic use

b) flux-local [FREE] — Run FLUX locally via ComfyUI
   Quality: Excellent | Speed: Medium (GPU required)
   Requires: ComfyUI installed + running, GPU with 8GB+ VRAM

c) replicate [PAID] — Replicate cloud models
   Models: FLUX 1.1 Pro, Imagen 4, Ideogram v3, FLUX Kontext
   Quality: Excellent | Cost: ~$0.003-0.05/image
   Requires: REPLICATE_API_TOKEN

d) kie [PAID] — KIE API image generation
   Quality: Good | Speed: Fast
   Requires: KIE_API_KEY
```

#### 3d. Video Generation

```
Video Provider:

a) google-ai-studio [FREE] — Google Veo via AI Studio
   Models: veo-3.1 (rate limited)
   Quality: Excellent | Speed: Medium | Cost: Free (rate limited)
   Requires: GOOGLE_AI_STUDIO_KEY (free to obtain)

b) cogvideo-local [FREE] — CogVideoX running locally
   Models: CogVideoX-2b, CogVideoX-5b
   Quality: Good | Speed: Slow
   Requires: GPU with 16GB+ VRAM, pip install diffusers

c) replicate [PAID] — Replicate cloud models
   Models: Wan 2.5, Kling 2.6 Pro
   Quality: Very good | Cost: ~$0.05-0.50/clip
   Requires: REPLICATE_API_TOKEN

d) kie [PAID] — Veo 3.1 via KIE API
   Quality: Excellent | Speed: Fast
   Requires: KIE_API_KEY
```

#### 3e. Subtitle/Transcription

```
Subtitle Provider:

a) faster-whisper [FREE] — CTranslate2-based Whisper, runs locally
   Models: tiny, base, small, medium, large-v3, large-v3-turbo
   Quality: Excellent (large-v3-turbo) | Speed: Fast
   Install: pip install faster-whisper

b) whisper-local [FREE] — Remotion's built-in Whisper.cpp
   Models: tiny, base, small, medium, large
   Quality: Good | Speed: Medium
   Install: npx remotion add @remotion/install-whisper-cpp

c) kie [PAID] — Cloud Whisper via KIE API
   Quality: Excellent | Speed: Fast | No local install
   Requires: KIE_API_KEY
```

#### 3f. Sound Effects

```
SFX Provider:

a) freesound [FREE] — Freesound.org library
   Thousands of free sound effects, search by keyword
   Quality: Varies | Speed: Instant (download)
   Note: Requires attribution (CC license)

b) elevenlabs (KIE) [PAID] — ElevenLabs SFX generation via KIE
   Custom AI-generated sound effects from text description
   Quality: Excellent | Speed: Fast
   Requires: KIE_API_KEY

c) elevenlabs-direct [PAID] — Direct ElevenLabs SFX
   Quality: Excellent | Speed: Fast
   Requires: ELEVENLABS_API_KEY
```

### 4. Write Configuration

After the user makes their choices, write the `config.yaml` file:

```bash
cat > config.yaml << 'YAML'
# Remotion Superpowers — Model Configuration
# Run /select-models to change these settings interactively

preset: [chosen_preset]

models:
  tts:
    provider: [chosen_tts]
    voice: [chosen_voice]
  music:
    provider: [chosen_music]
    model: [chosen_model]
  image:
    provider: [chosen_image]
    model: [chosen_model]
  video:
    provider: [chosen_video]
    model: [chosen_model]
  subtitle:
    provider: [chosen_subtitle]
    model: [chosen_model]
    device: [cpu_or_cuda]
  sfx:
    provider: [chosen_sfx]
YAML
```

### 5. Show Required Setup

Based on the selected configuration, show what needs to be installed:

#### API Keys Needed

```
API Keys Required for Your Configuration:
[List only the keys needed for chosen providers]

- GOOGLE_AI_STUDIO_KEY: https://aistudio.google.com/apikey (free)
- REPLICATE_API_TOKEN: https://replicate.com (paid per use)
- KIE_API_KEY: https://kie.ai (paid plan)
- ELEVENLABS_API_KEY: https://elevenlabs.io (free tier available)
- PEXELS_API_KEY: https://www.pexels.com/api/ (free)
```

#### Python Packages

```bash
# Install required packages for your selected providers
pip install [list of packages needed]
```

Common installs by provider:
- edge-tts: `pip install edge-tts`
- faster-whisper: `pip install faster-whisper`
- musicgen: `pip install transformers scipy soundfile torch`
- kokoro: `pip install kokoro soundfile`
- coqui-xtts: `pip install TTS`
- cogvideo: `pip install diffusers torch`

#### For the "free" preset, the full install command is:

```bash
pip install edge-tts faster-whisper transformers scipy soundfile torch
```

### 6. Verify Setup

Test that selected providers work:

```bash
# Test edge-tts
edge-tts --text "Hello, testing" --voice en-US-AriaNeural --write-media /tmp/test-tts.mp3 && echo "edge-tts: OK" || echo "edge-tts: FAILED"

# Test faster-whisper
python3 -c "from faster_whisper import WhisperModel; print('faster-whisper: OK')" 2>/dev/null || echo "faster-whisper: not installed"

# Test musicgen
python3 -c "from transformers import AutoProcessor; print('transformers: OK')" 2>/dev/null || echo "transformers: not installed"
```

### 7. Summary

Print the final configuration summary:

```
Model Configuration Complete!

  TTS:      [provider] — [cost]
  Music:    [provider] — [cost]
  Image:    [provider] — [cost]
  Video:    [provider] — [cost]
  Subtitle: [provider] — [cost]
  SFX:      [provider] — [cost]

  Estimated cost per video: [free / ~$X]
  API keys needed: [list or "none"]

  Config saved to: config.yaml
  
  You can change providers anytime with /select-models
  or by editing config.yaml directly.
```

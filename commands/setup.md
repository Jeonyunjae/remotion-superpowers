---
name: setup
description: First-run setup wizard for Remotion Superpowers. Checks dependencies, installs Remotion skills, walks through model provider selection and API key configuration, and verifies connections.
---

# Remotion Superpowers — Setup Wizard

You are running the setup wizard for the Remotion Superpowers plugin. Follow these steps carefully and guide the user through each one.

## Step 1: Check System Dependencies

Run these checks and report results:

```bash
node --version
npm --version
python3 --version 2>/dev/null || python --version 2>/dev/null
which uvx 2>/dev/null || echo "uvx not found"
pip3 --version 2>/dev/null || echo "pip not found"
```

**Required:**
- Node.js >= 18 (needed for Remotion and MCP servers)
- npm (comes with Node.js)

**Recommended:**
- Python 3 + `pip` (needed for free providers like edge-tts, faster-whisper)
- `uv`/`uvx` (needed for ElevenLabs and Pexels MCP servers)
  - If missing, tell user: `curl -LsSf https://astral.sh/uv/install.sh | sh`

Report each as pass or fail with version.

## Step 2: Check Remotion Project

Check if the current directory has a `remotion.config.ts` or `remotion.config.js` file.

**If YES:** Tell the user their Remotion project was detected.

**If NO:** Ask the user if they want to:
- Create a new Remotion project here: `npx create-video@latest`
  - Recommend: Blank template, yes to TailwindCSS, yes to install Skills
- Or navigate to an existing Remotion project first

Do NOT proceed until the user is in a Remotion project directory.

## Step 3: Install Remotion Agent Skills

Check if `.claude/skills/remotion-best-practices/` exists.

**If not installed:**
```bash
npx skills add remotion-dev/skills
```

Tell the user: "Remotion best practices skills installed. Claude now knows Remotion's APIs, animation patterns, audio handling, and more."

## Step 3.5: Choose Your Models

This is where you configure which AI providers to use for each function. This controls your costs.

Ask the user to choose:

```
How would you like to set up AI model providers?

1. FREE preset — All free/local providers ($0 cost)
   Uses: edge-tts, MusicGen, Pixazo, Google AI Studio, faster-whisper, Freesound
   You'll need: Python 3 + pip, Google AI Studio key (free)
   
2. BUDGET preset — Free where possible, paid for premium quality
   Uses: edge-tts, MusicGen, Replicate FLUX, Replicate Wan 2.5, faster-whisper, Freesound
   You'll need: REPLICATE_API_TOKEN (~$0.05-0.50/video)
   
3. PREMIUM preset — Best quality, all cloud services
   Uses: ElevenLabs, Suno, KIE images, Veo 3.1, KIE Whisper, ElevenLabs SFX
   You'll need: KIE_API_KEY (paid plan)
   
4. CUSTOM — Choose each provider individually
   Run /select-models for interactive selection
```

#### For FREE preset:

Install required Python packages:
```bash
pip install edge-tts faster-whisper transformers scipy soundfile torch
```

This installs:
- `edge-tts` — Microsoft neural TTS with Korean support
- `faster-whisper` — Local speech-to-text transcription
- `transformers` + `scipy` + `soundfile` — Meta's MusicGen for music generation
- `torch` — PyTorch backend for MusicGen

Estimated disk usage: ~2GB for models (downloaded on first use).

#### For BUDGET preset:

Install the same free packages plus configure Replicate:
```bash
pip install edge-tts faster-whisper transformers scipy soundfile torch
```

Then set the Replicate API token (see Step 4).

#### For PREMIUM preset:

No Python packages needed — all processing happens in the cloud via KIE.

#### Write config.yaml:

Based on the chosen preset, create the `config.yaml` file. Run `/select-models` if the user chose CUSTOM, or write the appropriate preset configuration directly.

## Step 4: Configure API Keys

Walk the user through ONLY the API keys needed for their chosen preset. Skip keys for services they won't use.

### For ALL presets:

#### Pexels API Key (FREE STOCK FOOTAGE)
- **What:** Free stock photos and videos. Search by keyword, download HD/4K clips directly into your project.
- **Signup:** https://www.pexels.com/api/ — completely free, create account and get key instantly
- **Variable:** `PEXELS_API_KEY`
- **Command:** `export PEXELS_API_KEY=your-key-here`
- **Persistence:** `echo 'export PEXELS_API_KEY=your-key-here' >> ~/.zshrc`

### For FREE preset only:

#### Google AI Studio Key (FREE VIDEO GENERATION)
- **What:** Access to Google Veo for video generation. Free with rate limits.
- **Signup:** https://aistudio.google.com/apikey — free, create and get key instantly
- **Variable:** `GOOGLE_AI_STUDIO_KEY`
- **Command:** `export GOOGLE_AI_STUDIO_KEY=your-key-here`
- **Persistence:** `echo 'export GOOGLE_AI_STUDIO_KEY=your-key-here' >> ~/.zshrc`

### For BUDGET preset (in addition to Pexels):

#### Replicate API Token (AI IMAGE/VIDEO MODELS)
- **What:** Access to FLUX for images, Wan 2.5/Kling for video generation. Pay-per-use pricing.
- **Signup:** https://replicate.com — create account, get API token from dashboard
- **Variable:** `REPLICATE_API_TOKEN`
- **Command:** `export REPLICATE_API_TOKEN=your-token-here`
- **Persistence:** `echo 'export REPLICATE_API_TOKEN=your-token-here' >> ~/.zshrc`

### For PREMIUM preset (in addition to Pexels):

#### KIE API Key (PRIMARY — covers TTS + Music + Image + Video + Subtitles + SFX)
- **What:** Single API key that provides music generation (Suno), sound effects (ElevenLabs SFX), text-to-speech (ElevenLabs TTS), image generation, video generation (Veo 3.1), and subtitle generation.
- **Signup:** https://kie.ai — create account, get API key from dashboard
- **Variable:** `KIE_API_KEY`
- **Command:** `export KIE_API_KEY=your-key-here`
- **Persistence:** `echo 'export KIE_API_KEY=your-key-here' >> ~/.zshrc`

### OPTIONAL for any preset:

#### TwelveLabs API Key (VIDEO UNDERSTANDING)
- **What:** Gives Claude the ability to "see" video files — analyze footage, find specific scenes, detect objects and speakers, generate summaries.
- **Signup:** https://www.twelvelabs.io — create account, get API key from dashboard
- **Variable:** `TWELVELABS_API_KEY`
- **Command:** `export TWELVELABS_API_KEY=your-key-here`
- **Note:** Optional. Only needed if you want to analyze existing video footage.

#### ElevenLabs API Key (ADVANCED VOICE)
- **What:** Direct access to ElevenLabs — voice cloning, custom voices, audio isolation.
- **Signup:** https://elevenlabs.io — free tier with 10k credits/month
- **Variable:** `ELEVENLABS_API_KEY`
- **Note:** Optional. Skip if you're using edge-tts or KIE for TTS.

After each key, ask the user to confirm they've set it before moving to the next one. It's OK to skip optional ones.

## Step 5: Reload Shell & Verify

Tell the user to reload their shell:
```bash
source ~/.zshrc
```

Then verify the environment variables are set (only check the ones relevant to their preset):
```bash
echo "PEXELS_API_KEY: ${PEXELS_API_KEY:+SET}"
echo "GOOGLE_AI_STUDIO_KEY: ${GOOGLE_AI_STUDIO_KEY:+SET}"
echo "KIE_API_KEY: ${KIE_API_KEY:+SET}" 
echo "TWELVELABS_API_KEY: ${TWELVELABS_API_KEY:+SET}"
echo "ELEVENLABS_API_KEY: ${ELEVENLABS_API_KEY:+SET}"
echo "REPLICATE_API_TOKEN: ${REPLICATE_API_TOKEN:+SET}"
```

## Step 6: Verify Connections

For the chosen preset, verify that tools work:

**Free provider verification:**
```bash
# Test edge-tts
edge-tts --text "Setup test" --voice en-US-AriaNeural --write-media /tmp/test-setup.mp3 && echo "edge-tts: OK" && rm /tmp/test-setup.mp3 || echo "edge-tts: FAILED"

# Test faster-whisper
python3 -c "from faster_whisper import WhisperModel; print('faster-whisper: OK')" 2>/dev/null || echo "faster-whisper: not installed"

# Test MusicGen
python3 -c "from transformers import AutoProcessor; print('transformers (MusicGen): OK')" 2>/dev/null || echo "transformers: not installed"
```

**MCP server verification:**
Tell the user to check MCP server status:
```
/mcp
```

All configured servers should show as connected. If any fail:
- Check that the API key environment variable is set
- For `uvx`-based servers, ensure `uv` is installed
- For `npx`-based servers, ensure Node.js is installed
- Try restarting Claude Code

## Step 7: Setup Complete

Print a summary:

```
Remotion Superpowers v2.1 — Setup Complete!

  Dependencies verified
  Remotion project detected
  Remotion skills installed
  Model providers configured: [preset name]
  API keys configured
  Connections verified

Provider Configuration:
  TTS:      [provider] — [free/paid]
  Music:    [provider] — [free/paid]
  Image:    [provider] — [free/paid]
  Video:    [provider] — [free/paid]
  Subtitle: [provider] — [free/paid]
  SFX:      [provider] — [free/paid]

Production Commands:
  /create-video      — Full video production from a prompt
  /create-short      — Short-form vertical video (TikTok/Reels/Shorts)

Asset Commands:
  /find-footage      — Search & download stock footage from Pexels
  /generate-image    — Generate AI images for your video
  /generate-clip     — Generate AI video clips

Audio Commands:
  /add-voiceover     — Generate and add narration
  /add-music         — Generate and add background music
  /transcribe        — Transcribe audio/video to text

Enhancement Commands:
  /add-captions      — TikTok-style animated captions
  /add-transitions   — Professional scene transitions

Analysis Commands:
  /analyze-footage   — Analyze existing video files with AI vision
  /review-video      — AI feedback loop on rendered output

Configuration:
  /select-models     — Change AI providers anytime

Try it: "Create a 30-second product demo video for a todo app"
```

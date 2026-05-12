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

## Step 3: 프로덕션 폴더 구조 생성

Remotion 프로젝트가 확인되면, 영상 제작에 필요한 폴더 구조를 자동으로 생성합니다. 이 구조는 전체 6-Phase 파이프라인의 산출물을 관리하며, 세션을 다시 시작해도 어디까지 진행했는지 파악할 수 있습니다.

```bash
# 산출물 관리 폴더
mkdir -p docs/storyboard-previews
mkdir -p docs/style-frames

# 미디어 에셋 폴더
mkdir -p public/audio
mkdir -p public/images
mkdir -p public/footage

# 렌더링 결과물 폴더
mkdir -p out/thumbnails
mkdir -p out/repurposed

# 납품 폴더
mkdir -p delivery
```

폴더 생성 후, 초기 진행 상황 문서를 작성합니다:

**파일**: `docs/00-progress.md`

```markdown
# 진행 상황 대시보드

> 마지막 업데이트: [현재 날짜 시간]
> `/progress` 명령으로 자동 업데이트됩니다.

## 전체 요약

| 항목 | 값 |
|------|---|
| 전체 진행률 | 0% |
| 필수 완료 | 0/16 |
| 선택 완료 | 0/18 |
| 현재 단계 | Phase 1 — 기획 |
| 다음 명령어 | /project-scope |

## Phase 1: 기획

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /project-scope | docs/00-project-scope.md | 필수 |
| ⬜ | /brand-kit | docs/00-brand-guidelines.md | 선택 |
| ⬜ | /select-models | config.yaml | 필수 |

## Phase 2: 프리프로덕션

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /receive-brief | docs/01-client-brief.md | 필수 |
| ⬜ | /creative-brief | docs/02-creative-brief.md | 필수 |
| ⬜ | /concept-options | docs/03-concepts.md | 선택 |
| ⬜ | /treatment | docs/03-treatment.md | 필수 ★승인 |
| ⬜ | /write-script | docs/04-script.md | 필수 ★승인 |
| ⬜ | /fact-check | docs/04-fact-check.md | 선택 |
| ⬜ | /storyboard | docs/05-storyboard.md | 필수 ★승인 |
| ⬜ | /style-frame | docs/style-frames/ | 선택 |
| ⬜ | /animatic | src/Animatic.tsx | 선택 |
| ⬜ | /prompt-sheet | docs/06-prompt-sheet.md | 필수 |
| ⬜ | /legal-check | docs/07-legal-checklist.md | 필수 ★승인 |

## Phase 3: 프로덕션

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /create-video | out/video.mp4 | 필수 |
| ⬜ | /add-voiceover | public/audio/voiceover.mp3 | 필수 |
| ⬜ | /add-music | public/audio/music.wav | 필수 |
| ⬜ | /generate-image | public/images/ | 선택 |
| ⬜ | /generate-clip | public/footage/ | 선택 |
| ⬜ | /find-footage | public/footage/ | 선택 |
| ⬜ | /add-captions | 자막 컴포넌트 | 선택 |
| ⬜ | /audio-mix | 오디오 최적화 | 선택 |
| ⬜ | /color-grade | 컬러 보정 | 선택 |

## Phase 4: 포스트프로덕션

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /review-video | 리뷰 피드백 | 필수 |
| ⬜ | /qc-check | QC 리포트 | 필수 |
| ⬜ | /revision-log | docs/08-revision-log.md | 선택 |
| ⬜ | /accessibility | 접근성 리포트 | 선택 |
| ⬜ | /thumbnail | out/thumbnails/ | 선택 |
| ⬜ | /seo-metadata | docs/10-seo-metadata.md | 선택 |

## Phase 5: 납품

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /export-multi | 플랫폼별 MP4 | 필수 |
| ⬜ | /deliver | docs/09-delivery-package.md | 필수 |
| ⬜ | /localize | 다국어 MP4 + SRT | 선택 |

## Phase 6: 공개 후

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ⬜ | /repurpose | out/repurposed/ | 선택 |
| ⬜ | /archive | docs/99-archive.md | 선택 |
```

사용자에게 안내:

> "프로덕션 폴더 구조를 생성했습니다:
> - `docs/` — 기획·프리프로덕션 문서 + 진행 상황
> - `public/audio|images|footage/` — 미디어 에셋
> - `out/` — 렌더링 결과물
> - `delivery/` — 납품 패키지
>
> `docs/00-progress.md`에서 전체 진행 상황을 확인할 수 있습니다.
> 아무 때나 `/progress`를 실행하면 자동으로 업데이트됩니다."

## Step 4: Install Remotion Agent Skills

Check if `.claude/skills/remotion-best-practices/` exists.


**If not installed:**
```bash
npx skills add remotion-dev/skills
```

Tell the user: "Remotion best practices skills installed. Claude now knows Remotion's APIs, animation patterns, audio handling, and more."

## Step 4.5: Choose Your Models

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

## Step 5: Configure API Keys

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

## Step 6: Reload Shell & Verify

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

## Step 7: Verify Connections

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

## Step 8: Setup Complete

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

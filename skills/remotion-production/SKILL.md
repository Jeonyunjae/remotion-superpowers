---
name: remotion-production
description: Full video production workflow for Remotion projects. Teaches how to orchestrate MCP tools (TTS, music, SFX, stock footage, video analysis) into complete Remotion compositions. Use this skill whenever producing a video that needs audio, voiceovers, music, stock footage, or analyzing existing video files.
---

# Remotion Production Workflow

This skill teaches how to produce complete videos with Remotion by orchestrating multiple MCP tools together. It covers the full pipeline from concept to rendered MP4.

## Model Provider System

Remotion Superpowers supports multiple AI providers for each media generation function. Users can choose between free local models and paid cloud APIs to control costs.

### How It Works

1. **`config.yaml`** in the project root stores the user's provider choices
2. Each command reads `config.yaml` to determine which provider to use
3. The `model-providers` rule contains detailed setup and usage for every provider
4. Users run `/select-models` to interactively configure providers, or edit `config.yaml` directly

### Available Presets

| Preset | Cost | Best For |
|---|---|---|
| free | $0 | Learning, prototyping, personal projects |
| budget | ~$0.05-0.50/video | Small teams, occasional production |
| premium | KIE plan pricing | Professional production, best quality |
| custom | Varies | Mix-and-match per function |

### Provider Overview by Function

| Function | Free Options | Paid Options |
|---|---|---|
| TTS | edge-tts, kokoro, coqui-xtts | elevenlabs (KIE), elevenlabs-direct |
| Music | musicgen, ace-step | suno (KIE) |
| Image | pixazo, flux-local | replicate, kie |
| Video | google-ai-studio, cogvideo-local | replicate, kie |
| Subtitle | faster-whisper, whisper-local | kie |
| SFX | freesound | elevenlabs (KIE), elevenlabs-direct |

Read `rules/model-providers.md` for full setup instructions, code examples, and quality comparisons for every provider.

### Adding New Providers

To add a new provider: update `rules/model-providers.md`, add the provider option to relevant command files, update `commands/select-models.md`, and add comments to `config.yaml`.

## Full Production Pipeline (39 Commands)

The complete video production pipeline covers 6 phases: Planning → Pre-production → Production → Post-production → Delivery → Post-release.

### Phase 1: Planning (기획)

| # | Command | Description | Output | Approval |
|---|---------|-------------|--------|----------|
| 1 | `/project-scope` | Define scope, deliverables, revisions, timeline, budget | `docs/00-project-scope.md` | - |
| 2 | `/brand-kit` | Import brand guidelines (logo, colors, fonts, tone) | `docs/00-brand-guidelines.md` | - |
| 3 | `/select-models` | Choose AI providers per function (free/paid) | `config.yaml` | - |

### Phase 2: Pre-production (프리프로덕션)

| # | Command | Description | Output | Approval |
|---|---------|-------------|--------|----------|
| 4 | `/receive-brief` | Receive and analyze client brief (10 questions) | `docs/01-client-brief.md` | - |
| 5 | `/creative-brief` | Create internal creative strategy (5 questions) | `docs/02-creative-brief.md` | - |
| 6 | `/concept-options` | Develop 2-3 creative concepts for comparison | `docs/03-concepts.md` | Optional |
| 7 | `/treatment` | Propose visual direction and style (6 questions) | `docs/03-treatment.md` | **Required** |
| 8 | `/write-script` | Write scene-by-scene AV script (5 questions) | `docs/04-script.md` | **Required** |
| 9 | `/fact-check` | Verify data accuracy in the script | `docs/04-fact-check.md` | Optional |
| 10 | `/storyboard` | Detail every shot with user Q&A (8 per scene) | `docs/05-storyboard.md` + previews | **Required** |
| 11 | `/style-frame` | Generate high-quality design reference frames | `docs/style-frames/` | Optional |
| 12 | `/animatic` | Create rough animated preview with timing | `src/Animatic.tsx` | Optional |
| 13 | `/prompt-sheet` | Convert storyboard to AI prompts | `docs/06-*.md` (3 files) | - |
| 14 | `/legal-check` | Verify legal compliance for all assets | `docs/07-legal-checklist.md` | **Required** |

### Phase 3: Production (프로덕션)

| # | Command | Description | Output |
|---|---------|-------------|--------|
| 15 | `/create-video` | Full production pipeline (6 sub-phases) | Remotion project + `out/video.mp4` |
| 16 | `/create-short` | Vertical short-form video pipeline | 9:16 MP4 |
| 17 | `/add-voiceover` | Generate and add TTS narration | `public/audio/voiceover.mp3` |
| 18 | `/add-music` | Generate and add background music | `public/audio/music.wav` |
| 19 | `/generate-image` | Generate AI images for scenes | `public/images/` |
| 20 | `/generate-clip` | Generate AI video clips | `public/footage/` |
| 21 | `/find-footage` | Search and download stock footage | `public/footage/` |
| 22 | `/add-captions` | Add TikTok-style animated captions | Caption component |
| 23 | `/add-transitions` | Add scene transitions | Transition effects |
| 24 | `/transcribe` | Transcribe audio to SRT | SRT file |
| 25 | `/audio-mix` | Apply LUFS standards, ducking, balance | Optimized audio |
| 26 | `/color-grade` | Check and fix color consistency | Color correction |
| 27 | `/analyze-footage` | Analyze existing video with AI | Analysis report |

### Phase 4: Post-production (포스트프로덕션)

| # | Command | Description | Output |
|---|---------|-------------|--------|
| 28 | `/review-video` | AI-powered video review (4 dimensions) | Review feedback |
| 29 | `/revision-log` | Track feedback rounds and version history | `docs/08-revision-log.md` |
| 30 | `/qc-check` | Technical QC (codec, LUFS, file size) | QC report |
| 31 | `/accessibility` | WCAG 2.1 AA compliance verification | Accessibility report |
| 32 | `/thumbnail` | Generate click-optimized thumbnails | `out/thumbnails/` |
| 33 | `/seo-metadata` | Generate titles, tags, descriptions | `docs/10-seo-metadata.md` |

### Phase 5: Delivery (납품)

| # | Command | Description | Output |
|---|---------|-------------|--------|
| 34 | `/export-multi` | Export to multiple platform formats | Platform-specific MP4s |
| 35 | `/deliver` | Create final delivery package | `docs/09-delivery-package.md` + `delivery/` |
| 36 | `/localize` | Create multilingual versions | Localized MP4s + SRTs |

### Phase 6: Post-release (공개 후)

| # | Command | Description | Output |
|---|---------|-------------|--------|
| 37 | `/repurpose` | Create derivative content (clips, GIF, blog) | `out/repurposed/` |
| 38 | `/archive` | Archive project with retrospective | `docs/99-archive.md` |
| 39 | `/setup` | Initialize Remotion project | Project scaffold |

### Document Flow

```
=== Planning ===
/project-scope → /brand-kit → /select-models

=== Pre-production ===
→ /receive-brief → /creative-brief → /concept-options (optional)
→ /treatment (approval) → /write-script (approval) → /fact-check (optional)
→ /storyboard (approval) → /style-frame (optional) → /animatic (optional)
→ /prompt-sheet → /legal-check (approval)

=== Production ===
→ /create-video (audio → visuals → composition → preview → render)
→ /audio-mix → /color-grade

=== Post-production ===
→ /review-video → /revision-log → /qc-check → /accessibility
→ /thumbnail → /seo-metadata

=== Delivery ===
→ /export-multi → /deliver → /localize (optional)

=== Post-release ===
→ /repurpose → /archive
```

### Key Principle: Ask, Don't Assume

All commands follow the **Questioning Protocol** (`rules/questioning-protocol.md`):

1. **Never assume** — ask for every piece of missing information
2. **Detect ambiguity** — vague answers like "알아서", "적당히" trigger follow-up questions with concrete options
3. **Progressive depth** — start with big-picture questions, then drill into details
4. **Always offer choices** — present 2-4 concrete options when the user is unsure
5. **Validate completeness** — verify all required fields before writing any document

The agent keeps asking until it has production-ready information.

### Production Integration

`/create-video` and `/create-short` check for pre-production documents before starting:
- `docs/04-script.md` for narration text (Phase 2: Audio)
- `docs/06-prompt-sheet.md` for AI generation prompts (Phase 3: Visuals)
- `docs/05-storyboard.md` for scene structure (Phase 4: Composition)
- `docs/00-brand-guidelines.md` for brand consistency (if exists)
- `docs/07-legal-checklist.md` for legal compliance (if exists)

---

## Available MCP Tools

You have access to these MCP servers for media production (used by paid/cloud providers):

### remotion-media (via KIE)
- `generate_tts` — Text-to-speech voiceovers (ElevenLabs TTS)
- `generate_music` — Background music (Suno V3.5-V5)
- `generate_sfx` — Sound effects (ElevenLabs SFX V2)
- `generate_image` — AI images (Nano Banana Pro)
- `generate_video` — AI video clips (Veo 3.1)
- `generate_subtitles` — Transcribe audio/video to SRT (Whisper)
- `list_assets` — List all generated media in the project

### TwelveLabs (video understanding)
- Index and analyze video files
- Semantic search within videos ("find the part where...")
- Scene detection, object detection, speaker identification
- Video summarization

### Pexels (stock footage)
- `searchPhotos` — Search free stock photos
- `searchVideos` — Search free stock videos
- `getVideo` / `getPhoto` — Get details by ID
- `downloadVideo` — Download video to project

### ElevenLabs (optional — advanced voice)
- Voice cloning from audio samples
- Advanced TTS with custom voices
- Audio isolation and processing
- Transcription

### Replicate (optional — 100+ AI models)
- `replicate_run` — Run a model synchronously (images)
- `replicate_create_prediction` — Start async prediction (video)
- `replicate_get_prediction` — Poll prediction status
- Image models: FLUX 1.1 Pro, Imagen 4, Ideogram v3, FLUX Kontext
- Video models: Wan 2.5 (T2V, I2V), Kling 2.6 Pro

## Production Pipeline

Read individual rule files for detailed workflows:

- `rules/production-pipeline.md` — End-to-end workflow from concept to final render
- `rules/audio-integration.md` — How to integrate generated audio into Remotion compositions
- `rules/voiceover-sync.md` — Syncing TTS voiceovers with animations and captions
- `rules/music-scoring.md` — Generating and timing background music
- `rules/stock-footage-workflow.md` — Searching, downloading, and using stock footage in Remotion
- `rules/video-analysis.md` — Using TwelveLabs to analyze and select clips from existing footage
- `rules/captions-workflow.md` — TikTok-style animated captions using @remotion/captions and Whisper
- `rules/animation-presets.md` — Reusable animation patterns (fade, slide, scale, typewriter, stagger)
- `rules/3d-content.md` — Three.js and React Three Fiber via @remotion/three
- `rules/data-visualization.md` — Animated charts, dashboards, and number counters
- `rules/visual-effects.md` — Light leaks, Lottie, film grain, vignettes, Ken Burns
- `rules/ci-rendering.md` — GitHub Actions workflows for automated video rendering
- `rules/replicate-models.md` — Replicate MCP model catalog, usage, and decision guide
- `rules/image-generation.md` — AI image prompt engineering, provider selection, Remotion integration
- `rules/video-generation.md` — AI video clip generation, I2V pipeline, sequencing in Remotion
- `rules/sound-effects.md` — SFX generation, prompt engineering, timing to visual events
- `rules/elevenlabs-advanced.md` — Voice cloning, custom TTS parameters, multi-voice scripts
- `rules/asset-management.md` — File organization, naming conventions, staticFile() reference
- `rules/model-providers.md` — All supported providers, setup instructions, usage examples, cost/quality comparison
- `rules/questioning-protocol.md` — Ambiguity detection, follow-up questioning, "알아서 해줘" handling

## Key Principles

1. **Audio drives timing** — Generate voiceover first, get its duration, then set composition length to match.
2. **Assets go in `public/`** — All generated media files (audio, video, images) must be saved to the project's `public/` directory so Remotion can access them via `staticFile()`.
3. **Use Remotion's audio components** — Always use `<Audio>` component with `staticFile()` for audio. Never use HTML `<audio>` tags.
4. **Frame-based timing** — Remotion uses frames, not seconds. Convert with `fps * seconds`. At 30fps, 1 second = 30 frames.
5. **Progressive composition** — Build the video in layers: visuals first, then voiceover, then music, then SFX.
6. **Preview frequently** — Use `npm run dev` to preview after each major change. The Remotion player updates live.

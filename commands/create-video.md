---
name: create-video
description: Full video production pipeline with sub-agent parallelization. Orchestrates media-scout (visual assets) and post-producer (review/QC) as sub-agents for faster production.
---

# Create Video — 전체 영상 제작 파이프라인

오케스트레이터로서 전체 파이프라인을 총괄합니다. 비주얼 에셋 수집은 **media-scout**에, 포스트 검증은 **post-producer**에 위임하여 병렬로 처리합니다.

**참조 규칙:**
- `rules/agent-orchestration.md` — 서브에이전트 병렬 처리 구조
- `rules/progress-tracking.md` — 진행 상황 자동 갱신
- `rules/production-pipeline.md` — E2E 파이프라인 상세
- `rules/model-providers.md` — 프로바이더별 사용법

## 파이프라인 구조

```
Phase 1: 문서 확인                    [순차 — 직접]
Phase 2: 오디오 생성                  [순차→병렬 — 직접]
Phase 3: 비주얼 에셋 수집             [병렬 — media-scout 위임]  ←── 핵심
Phase 4: Remotion 컴포지션 작성       [순차 — 직접]
Phase 5: 프리뷰 + 피드백              [순차 — 직접]
Phase 6: 렌더링                      [순차 — 직접]
Phase 7: 포스트프로덕션 검증           [병렬 — post-producer 위임]  ←── 핵심
Phase 8: 수정 루프                    [순차 — 직접]
```

---

## Phase 1: 문서 확인 + 설정 [순차 — 직접]

### 프리프로덕션 문서 체크

```bash
ls docs/01-client-brief.md docs/02-creative-brief.md docs/03-treatment.md docs/04-script.md docs/05-storyboard.md docs/06-prompt-sheet.md 2>&1
```

**하나라도 없으면 중단:**
> "프리프로덕션 문서가 미완성입니다. 다음 커맨드를 먼저 실행해주세요:
> `/receive-brief` → `/creative-brief` → `/treatment` → `/write-script` → `/storyboard` → `/prompt-sheet`"

### 설정 확인

```bash
cat config.yaml 2>/dev/null
```

프로바이더 확인: TTS, Music, Image, Video, Subtitle, SFX 각각 어떤 프로바이더인지 파악.
없으면 `/select-models` 실행 안내 또는 무료 기본값 사용.

---

## Phase 2: 오디오 생성 [순차→병렬 — 직접]

### 2a. 나레이션 생성 (순차 — 반드시 먼저)

`docs/04-script.md`에서 "전체 나레이션 스크립트 (연결)" 섹션 추출.

config.yaml의 TTS 프로바이더에 따라 생성:

**edge-tts:**
```bash
edge-tts --text "[나레이션 전문]" --voice "[voice]" --write-media public/audio/voiceover.mp3
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.mp3
```

**elevenlabs (KIE):**
```
remotion-media generate_tts:
  text: [나레이션 전문]
  voice: [선택된 목소리]
  project_path: [프로젝트 루트]
```

**나레이션 duration을 기록 — 이것이 전체 영상 길이의 기준.**

### 2b + 2c. 배경음악 + 효과음 (병렬)

나레이션 duration이 확정되면, 음악과 효과음을 **동시에** 생성합니다:

**배경음악** (config.yaml 프로바이더):
```
musicgen / suno 등으로 생성
prompt에 반드시 "no vocals" 포함
duration: 나레이션 길이에 맞춤
→ public/audio/music.wav
```

**효과음** (config.yaml 프로바이더):
```
freesound 검색 또는 elevenlabs 생성
docs/04-script.md의 효과음 지시 참조
→ public/audio/sfx/[effect-name].mp3
```

---

## Phase 3: 비주얼 에셋 수집 [병렬 — media-scout 위임]

**media-scout 서브에이전트를 생성하여 위임합니다.**

```
media-scout에게 전달:

"docs/06-prompt-sheet.md, docs/06-style-guide.md, docs/06-model-selection.md를 읽고
config.yaml의 프로바이더 설정에 따라 다음 에셋을 병렬로 수집해줘:

1. 스톡 영상: 프롬프트 시트에서 'stock' 지정된 장면 → Pexels 검색 → public/footage/
2. AI 이미지: 프롬프트 시트에서 'image' 지정된 장면 → [image provider] 생성 → public/images/
3. AI 영상 클립: 프롬프트 시트에서 'video' 지정된 장면 → [video provider] 생성 → public/footage/

스토리보드(docs/05-storyboard.md) 참조하여 장면 순서와 용도를 파악할 것.
수집 완료 후 에셋 경로 목록과 실패 항목을 보고해줘."
```

**media-scout가 에셋을 수집하는 동안**, Phase 2b/2c의 음악·효과음이 아직 진행 중이면 함께 병렬로 대기합니다.

모든 에셋 준비 완료 후 Phase 4로 진행.

---

## Phase 4: Remotion 컴포지션 작성 [순차 — 직접]

`docs/05-storyboard.md` 기반으로 Remotion 코드 작성:

1. **씬 컴포넌트** — `src/scenes/Scene01.tsx`, `Scene02.tsx`, ...
2. **메인 컴포지션** — 씬 시퀀싱, `<Sequence>` 타이밍
3. **오디오 레이어** — 나레이션 + 음악(더킹) + 효과음
4. **컴포지션 등록** — `src/Root.tsx`에 duration 설정

```tsx
<Composition
  id="MainVideo"
  component={MainVideo}
  durationInFrames={Math.ceil(voiceoverDuration * 30) + 60}
  fps={30}
  width={1920}
  height={1080}
/>
```

---

## Phase 5: 프리뷰 + 피드백 [순차 — 직접]

```bash
npm run dev
```

> "프리뷰를 확인해주세요 (http://localhost:3000). 수정이 필요한 부분이 있으면 말씀해주세요."

피드백 반영 → 재프리뷰 반복.

---

## Phase 6: 렌더링 [순차 — 직접]

```bash
npx remotion render MainVideo out/video.mp4 --codec h264 --crf 18
```

---

## Phase 7: 포스트프로덕션 검증 [병렬 — post-producer 위임]

**post-producer 서브에이전트를 생성하여 위임합니다.**

```
post-producer에게 전달:

"out/video.mp4가 렌더링 완료되었습니다. 다음 3가지를 병렬로 검증해줘:

1. AI 리뷰: 비주얼 품질, 페이싱, 오디오-비주얼 싱크, 플랫폼 적합성
   참조: docs/03-treatment.md (비주얼 방향), docs/05-storyboard.md (의도한 구성)

2. QC 기술 검증: ffprobe로 코덱/해상도/FPS/LUFS/파일크기/오디오코덱
   기준: H.264, 1920x1080, 30fps, -14 LUFS (YouTube)

3. 접근성: 자막 존재, 색상대비 4.5:1, 깜빡임 <3회/초, 오디오 밸런스

종합 리포트(점수/판정/수정항목)를 보고해줘."
```

---

## Phase 8: 수정 루프 [순차 — 직접]

post-producer 리포트 기반:

- **APPROVED (8+/10)**: 완료 → `/progress` 갱신
- **NEEDS REVISION**: CRITICAL/MAJOR 이슈 수정 → Phase 6(재렌더링) → Phase 7(재검증)

수정 루프는 APPROVED 될 때까지 반복합니다.

---

## 완료 시

> "영상 제작이 완료되었습니다.
> 📁 out/video.mp4
> 📊 진행 상황: 필수 [n/16] 완료 | 다음: /[커맨드명]"

`docs/00-progress.md` 자동 갱신 (`rules/progress-tracking.md` 적용).

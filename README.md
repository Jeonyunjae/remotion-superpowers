# Remotion Superpowers (Custom Fork)

> AI 영상 제작 자동화 패키지 — Claude Code + Remotion 기반  
> 원본: [DojoCodingLabs/remotion-superpowers](https://github.com/DojoCodingLabs/remotion-superpowers) (MIT)  
> 수정: AI 모델 선택 시스템 추가 (무료/저비용/프리미엄 전환 가능)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

---

## 개요

텍스트 프롬프트 하나로 **기획 → 스크립트 → 이미지/영상 생성 → 음성 → 음악 → 자막 → 편집 → 렌더링**까지 전 과정을 자동화하는 Claude Code 플러그인입니다.

**원본과의 차이점**: 원본은 KIE(유료 API)에 고정되어 있지만, 이 Fork는 기능별로 **무료/유료 AI 모델을 사용자가 선택**할 수 있습니다.

---

## 영상 제작 프로세스

### 전체 흐름도

```
사용자 프롬프트 입력
    |
    v
Phase 1. 기획 (Concept Breakdown)
    |   - 장면 분할, 러닝타임 결정
    |   - 오디오/비주얼 계획 수립
    |   - 사용자 승인
    v
Phase 2. 오디오 생성 (Audio First)       ← 오디오가 전체 타이밍을 결정
    |   - 2a. 나레이션 (TTS)
    |   - 2b. 배경음악 (Music)
    |   - 2c. 효과음 (SFX)
    v
Phase 3. 비주얼 에셋 확보 (Visual Assets)
    |   - 3a. 스톡 영상 검색 (Pexels)
    |   - 3b. AI 이미지 생성
    |   - 3c. AI 영상 클립 생성
    |   - 3d. 기존 영상 분석 (선택)
    v
Phase 4. Remotion 코드 생성 (Composition)
    |   - React/TSX 컴포넌트 작성
    |   - 장면별 시퀀스 구성
    |   - 오디오 레이어 연결
    |   - 애니메이션/전환효과 적용
    v
Phase 5. 자막 생성 (Captions)
    |   - Whisper로 음성 → 텍스트 변환
    |   - TikTok 스타일 애니메이션 자막
    v
Phase 6. 프리뷰 & 수정 (Preview)
    |   - localhost:3000에서 실시간 확인
    |   - 사용자 피드백 반영
    v
Phase 7. 렌더링 (Render)
    |   - npx remotion render → MP4 출력
    v
Phase 8. AI 리뷰 (Review Loop)        ← 선택적 반복
    |   - AI가 영상을 분석하고 피드백
    |   - 수정 → 재렌더 → 리뷰 반복
    |   - 8점 이상 달성 시 완료
    v
완성된 MP4 파일
```

---

## 프로세스별 업무 & AI 모델 매핑

### Phase 1. 기획 (Concept Breakdown)

| 업무 | 담당 | 설명 |
|------|------|------|
| 프롬프트 분석 | Claude | 사용자 요청을 분석하여 영상 구조 도출 |
| 장면 분할 | Claude | 시간대별 장면 구성 (Intro → 본문 → Outro) |
| 오디오 계획 | Claude | 나레이션 여부, 음악 장르, 효과음 목록 |
| 비주얼 계획 | Claude | 장면별 필요 에셋 (스톡/AI생성/기존) |
| 사용자 승인 | 사용자 | 기획안 확인 후 진행 승인 |

> 이 단계에서 외부 AI는 사용하지 않음. Claude가 스크립트와 구조를 생성.

---

### Phase 2. 오디오 생성 (Audio First)

**왜 오디오가 먼저인가?** 나레이션 길이가 전체 영상의 러닝타임을 결정하기 때문.

#### 2a. 나레이션 (TTS)

| 프로바이더 | 비용 | 한국어 | 음성 수 | 품질 |
|-----------|------|--------|---------|------|
| **edge-tts** | 무료 | O (9개 음성) | 300+ | 높음 |
| **kokoro** | 무료 | X | 8 | 높음 |
| **coqui-xtts** | 무료 | O | 무제한 (클로닝) | 높음 |
| **ElevenLabs (KIE)** | 유료 | X | 7 | 매우 높음 |
| **ElevenLabs (직접)** | 유료 | X | 전체 + 클로닝 | 매우 높음 |

```
작동 방식 (edge-tts 예시):
  스크립트 텍스트 → edge-tts CLI → public/audio/voiceover.mp3 → <Audio> 컴포넌트 연결
```

#### 2b. 배경음악 (Music)

| 프로바이더 | 비용 | 생성 시간 | 품질 | GPU 필요 |
|-----------|------|----------|------|---------|
| **MusicGen (Meta)** | 무료 | 30초~2분 | 좋음 | 권장 |
| **ACE-Step** | 무료 | 2초 (GPU) | 매우 좋음 | 필수 |
| **Suno (KIE)** | 유료 | 빠름 | 최고 | 불필요 |

```
작동 방식 (MusicGen 예시):
  "calm lo-fi, no vocals" 프롬프트 → MusicGen 모델 → public/audio/music.wav → 페이드인/아웃 적용
```

#### 2c. 효과음 (SFX)

| 프로바이더 | 비용 | 방식 | 품질 |
|-----------|------|------|------|
| **Freesound** | 무료 | 라이브러리 검색 | 다양 |
| **ElevenLabs (KIE)** | 유료 | AI 생성 | 높음 |

---

### Phase 3. 비주얼 에셋 확보

#### 3a. 스톡 영상 (Pexels)

| 프로바이더 | 비용 | 해상도 | 제한 |
|-----------|------|--------|------|
| **Pexels** | 완전 무료 | HD/4K | 월 20,000 요청 |

```
작동 방식:
  키워드 검색 → 결과 제시 → 선택 다운로드 → public/footage/ 저장
```

#### 3b. AI 이미지 생성

| 프로바이더 | 비용 | 모델 | 품질 | GPU 필요 |
|-----------|------|------|------|---------|
| **Pixazo** | 무료 | FLUX Schnell | 좋음 | 불필요 |
| **FLUX 로컬** | 무료 | FLUX.1 | 높음 | 12GB+ |
| **Replicate** | ~$0.003/장 | FLUX Pro, Imagen 4, Ideogram v3 | 매우 높음 | 불필요 |
| **KIE** | 유료 | Nano Banana Pro | 좋음 | 불필요 |

```
작동 방식 (Pixazo 예시):
  이미지 프롬프트 → Pixazo API → base64 디코딩 → public/images/ 저장 → <Img> 컴포넌트
```

#### 3c. AI 영상 클립 생성

| 프로바이더 | 비용 | 모델 | 클립 길이 | GPU 필요 |
|-----------|------|------|----------|---------|
| **Google AI Studio** | 무료 | Veo 3.1 | 3~8초 | 불필요 |
| **CogVideo 로컬** | 무료 | CogVideoX-2B | 3~6초 | 16GB+ |
| **Replicate** | ~$0.05/클립 | Wan 2.5, Kling 2.6 | 3~10초 | 불필요 |
| **KIE** | 유료 | Veo 3.1 | 3~8초 | 불필요 |

```
작동 방식:
  - Text-to-Video: 텍스트 프롬프트 → AI 모델 → MP4 클립
  - Image-to-Video: 정지 이미지 → AI 모델 → 움직이는 영상 클립
```

#### 3d. 기존 영상 분석 (TwelveLabs)

| 프로바이더 | 비용 | 기능 |
|-----------|------|------|
| **TwelveLabs** | 무료 티어 | 장면 탐지, 객체 인식, 시맨틱 검색, 요약 |

```
작동 방식:
  기존 영상 업로드 → 인덱싱 → "커피 따르는 장면 찾아줘" → 타임스탬프 반환
```

---

### Phase 4. Remotion 코드 생성

| 업무 | 담당 | 설명 |
|------|------|------|
| 장면 컴포넌트 생성 | Claude | src/scenes/에 React/TSX 파일 생성 |
| 메인 컴포지션 | Claude | 장면 시퀀스 + 타이밍 구성 |
| 오디오 레이어 | Claude | 나레이션 + BGM + SFX 볼륨 조절 (더킹) |
| 애니메이션 | Claude | interpolate(), spring() 기반 모션 |
| 전환 효과 | Claude | fade, slide, wipe, flip, clockWipe |

```
생성되는 코드 구조:
  src/
  ├── Root.tsx           ← 컴포지션 등록 (해상도, fps, 길이)
  ├── MyVideo.tsx        ← 메인 영상 컴포지션
  ├── scenes/
  │   ├── Intro.tsx      ← 장면별 컴포넌트
  │   ├── Scene1.tsx
  │   └── Outro.tsx
  └── components/
      ├── AnimatedText.tsx
      └── AudioLayer.tsx
```

> 이 단계에서 외부 AI는 사용하지 않음. Claude가 Remotion React 코드를 직접 작성.

---

### Phase 5. 자막 생성

| 프로바이더 | 비용 | 모델 | 속도 | GPU 필요 |
|-----------|------|------|------|---------|
| **faster-whisper** | 무료 | large-v3-turbo | 빠름 | 권장 |
| **whisper-local** | 무료 | medium | 보통 | 권장 |
| **KIE** | 유료 | Whisper API | 빠름 | 불필요 |

```
작동 방식:
  나레이션 MP3 → Whisper → 단어별 타임스탬프 → SRT/JSON 생성
  → @remotion/captions → TikTok 스타일 애니메이션 자막
```

---

### Phase 6~7. 프리뷰 & 렌더링

| 업무 | 명령어 | 설명 |
|------|--------|------|
| 실시간 프리뷰 | `npm run dev` | localhost:3000에서 영상 미리보기 |
| 최종 렌더링 | `npx remotion render` | MP4 파일 출력 |
| 고품질 렌더링 | `npx remotion render --crf 18` | 낮은 CRF = 높은 품질 |

---

### Phase 8. AI 리뷰 루프 (선택)

| 평가 차원 | 분석 내용 |
|-----------|----------|
| 비주얼 품질 | 구도, 텍스트 가독성, 전환 부드러움, 빈 프레임 검사 |
| 페이싱 | 장면 길이, 내러티브 흐름, 전환 타이밍 |
| 오디오-비주얼 싱크 | 나레이션↔화면 일치, SFX 타이밍, BGM 적합성 |
| 플랫폼 적합성 | 화면비, 세이프존, 길이, 캡션 유무 |

```
평가 기준:
  9~10점: 퍼블리싱 가능
  7~8점: 약간의 수정 필요
  5~6점: 눈에 띄는 이슈 다수
  1~4점: 대폭 수정 필요

  → 8점 이상 or CRITICAL/MAJOR 이슈 제로 시 완료
```

---

## AI 모델 선택 시스템

### 비용 프리셋 3종

| 프리셋 | 월 비용 | TTS | 음악 | 이미지 | 영상 | 자막 | SFX |
|--------|---------|-----|------|--------|------|------|-----|
| **free** | **$0** | edge-tts | MusicGen | Pixazo | Google AI Studio | faster-whisper | Freesound |
| **budget** | ~$1~5/영상 | edge-tts | MusicGen | Replicate | Replicate | faster-whisper | Freesound |
| **premium** | KIE 요금제 | ElevenLabs | Suno | KIE | Veo 3.1 | KIE Whisper | ElevenLabs |

### 프리셋 전환

```bash
# Claude Code 내에서:
/select-models          # 대화형 모델 선택

# 또는 config.yaml 직접 수정:
preset: free            # free | budget | premium | custom
```

### config.yaml 구조

```yaml
preset: free

models:
  tts:
    provider: edge-tts
    voice: ko-KR-SunHiNeural
  music:
    provider: musicgen
    model: facebook/musicgen-small
  image:
    provider: pixazo
    model: flux-schnell
  video:
    provider: google-ai-studio
    model: veo-3.1
  subtitle:
    provider: faster-whisper
    model: large-v3-turbo
    device: cpu
  sfx:
    provider: freesound
```

---

## 전체 프로바이더 목록

### TTS (나레이션)

| 프로바이더 | 비용 | 한국어 | 설치 | 특징 |
|-----------|------|--------|------|------|
| edge-tts | 무료 | O | `pip install edge-tts` | MS 뉴럴 음성 300+개 |
| kokoro | 무료 | X | `pip install kokoro soundfile` | 82M 초경량, 고품질 |
| coqui-xtts | 무료 | O | `pip install TTS` | 음성 클로닝 가능 |
| elevenlabs (KIE) | 유료 | X | API 키만 | 최고 품질 |
| elevenlabs-direct | 유료 | X | API 키만 | 클로닝 + 전체 기능 |

### 음악 (BGM)

| 프로바이더 | 비용 | 설치 | 특징 |
|-----------|------|------|------|
| musicgen | 무료 | `pip install transformers scipy soundfile` | Meta 오픈소스, CPU 가능 |
| ace-step | 무료 | 별도 설치 (GPU 필수) | 상용급 품질, REST API |
| suno (KIE) | 유료 | API 키만 | 프로 수준 음악 |

### 이미지 생성

| 프로바이더 | 비용 | 설치 | 특징 |
|-----------|------|------|------|
| pixazo | 무료 | 없음 (클라우드 API) | FLUX 기반, 즉시 사용 |
| flux-local | 무료 | ComfyUI + GPU 12GB+ | 최고 품질, 로컬 |
| replicate | ~$0.003/장 | API 키만 | FLUX Pro, Imagen 4 등 |
| kie | 유료 | API 키만 | 통합 API |

### 영상 생성

| 프로바이더 | 비용 | 설치 | 클립 길이 | 특징 |
|-----------|------|------|----------|------|
| google-ai-studio | 무료 | API 키 (무료) | 3~8초 | Veo 3.1, 속도 제한 |
| cogvideo-local | 무료 | `pip install diffusers` + GPU 16GB+ | 3~6초 | CogVideoX-2B |
| replicate | ~$0.05/클립 | API 키만 | 3~10초 | Wan 2.5, Kling 2.6 |
| kie | 유료 | API 키만 | 3~8초 | Veo 3.1 통합 |

### 자막

| 프로바이더 | 비용 | 설치 | 특징 |
|-----------|------|------|------|
| faster-whisper | 무료 | `pip install faster-whisper` | 4배 빠른 Whisper, CPU 가능 |
| whisper-local | 무료 | `npx remotion add @remotion/install-whisper-cpp` | Remotion 내장 |
| kie | 유료 | API 키만 | 클라우드, 설치 불필요 |

### 효과음 (SFX)

| 프로바이더 | 비용 | 방식 | 특징 |
|-----------|------|------|------|
| freesound | 무료 | 라이브러리 검색 | CC 라이선스, 크레딧 필요 |
| elevenlabs (KIE) | 유료 | AI 생성 | 텍스트에서 맞춤 효과음 |

---

## 필요 API 키 (프리셋별)

| API 키 | free | budget | premium | 발급처 |
|--------|------|--------|---------|--------|
| GOOGLE_AI_STUDIO_KEY | 필요 (무료) | - | - | https://aistudio.google.com/apikey |
| PEXELS_API_KEY | 권장 (무료) | 권장 (무료) | 권장 (무료) | https://www.pexels.com/api/ |
| REPLICATE_API_TOKEN | - | 필요 (유료) | - | https://replicate.com |
| KIE_API_KEY | - | - | 필요 (유료) | https://kie.ai |
| TWELVELABS_API_KEY | 선택 | 선택 | 필요 | https://www.twelvelabs.io |
| ELEVENLABS_API_KEY | - | - | 선택 | https://elevenlabs.io |

---

## 설치

### 필수 요건
- Claude Code (플러그인 지원 버전)
- Node.js >= 18
- Python 3 + pip (무료 프로바이더 사용 시)

### 설치 방법

```bash
# 1. 레포 클론
git clone https://github.com/Jeonyunjae/remotion-superpowers.git
cd remotion-superpowers

# 2. Claude Code에서 플러그인 등록
/plugin marketplace add .
/plugin install remotion-superpowers

# 3. 설정 마법사 실행
/setup
```

### 무료 프리셋 빠른 설치

```bash
# Python 패키지 설치 (무료 프로바이더용)
pip install edge-tts faster-whisper transformers scipy soundfile torch
```

---

## 커맨드 목록 (14개)

### 제작

| 커맨드 | 설명 |
|--------|------|
| `/setup` | 초기 설정 마법사 (의존성 + 모델 선택 + API 키) |
| `/select-models` | **[신규]** AI 모델 프로바이더 선택 (무료/유료 전환) |
| `/create-video` | 전체 영상 제작 파이프라인 (프롬프트 → MP4) |
| `/create-short` | 숏폼 세로 영상 (TikTok/Reels/Shorts) |

### 에셋

| 커맨드 | 설명 |
|--------|------|
| `/find-footage` | Pexels 스톡 영상/사진 검색 + 다운로드 |
| `/generate-image` | AI 이미지 생성 (장면 배경, 썸네일 등) |
| `/generate-clip` | AI 영상 클립 생성 (Text-to-Video, Image-to-Video) |

### 오디오

| 커맨드 | 설명 |
|--------|------|
| `/add-voiceover` | 나레이션 생성 + 컴포지션 연결 |
| `/add-music` | 배경음악 생성 + 페이드인/아웃 + 더킹 |
| `/transcribe` | 오디오/비디오 → 텍스트 변환 |

### 편집

| 커맨드 | 설명 |
|--------|------|
| `/add-captions` | TikTok 스타일 애니메이션 자막 |
| `/add-transitions` | 장면 전환 효과 (fade, slide, wipe, flip) |

### 분석

| 커맨드 | 설명 |
|--------|------|
| `/analyze-footage` | 기존 영상 AI 분석 (장면탐지, 객체인식) |
| `/review-video` | AI 리뷰 피드백 루프 (렌더→리뷰→수정 반복) |

---

## 프로젝트 구조

```
remotion-superpowers/
├── config.yaml                          [신규] AI 모델 선택 설정
├── .mcp.json                            5개 MCP 서버 설정
├── .claude-plugin/
│   ├── plugin.json                      플러그인 메타데이터
│   └── marketplace.json                 마켓플레이스 등록 정보
├── commands/                            14개 슬래시 커맨드
│   ├── setup.md                         초기 설정 마법사
│   ├── select-models.md                 [신규] 모델 선택
│   ├── create-video.md                  전체 영상 파이프라인
│   ├── create-short.md                  숏폼 영상
│   ├── find-footage.md                  스톡 영상 검색
│   ├── generate-image.md                AI 이미지 생성
│   ├── generate-clip.md                 AI 영상 생성
│   ├── add-voiceover.md                 나레이션 생성
│   ├── add-music.md                     배경음악 생성
│   ├── add-captions.md                  자막 생성
│   ├── add-transitions.md               전환 효과
│   ├── transcribe.md                    음성→텍스트
│   ├── analyze-footage.md               영상 분석
│   └── review-video.md                  AI 리뷰 루프
├── agents/                              3개 AI 에이전트
│   ├── video-director.md                전체 파이프라인 오케스트레이션
│   ├── media-scout.md                   미디어 에셋 탐색/평가
│   └── post-producer.md                 리뷰/반복 개선
├── skills/
│   ├── remotion-production/
│   │   ├── SKILL.md                     프로덕션 워크플로우 총괄
│   │   └── rules/                       19개 프로덕션 규칙
│   │       ├── model-providers.md       [신규] 프로바이더 상세 가이드
│   │       ├── production-pipeline.md   E2E 파이프라인
│   │       ├── audio-integration.md     오디오 통합
│   │       ├── voiceover-sync.md        나레이션 싱크
│   │       ├── music-scoring.md         음악 스코어링
│   │       ├── stock-footage-workflow.md 스톡 영상 워크플로우
│   │       ├── video-analysis.md        영상 분석
│   │       ├── captions-workflow.md     자막 워크플로우
│   │       ├── animation-presets.md     애니메이션 프리셋
│   │       ├── 3d-content.md            3D 콘텐츠
│   │       ├── data-visualization.md    데이터 시각화
│   │       ├── visual-effects.md        비주얼 이펙트
│   │       ├── ci-rendering.md          CI/CD 렌더링
│   │       ├── replicate-models.md      Replicate 모델
│   │       ├── image-generation.md      이미지 생성
│   │       ├── video-generation.md      영상 생성
│   │       ├── sound-effects.md         효과음
│   │       ├── elevenlabs-advanced.md   ElevenLabs 고급
│   │       └── asset-management.md      에셋 관리
│   └── setup-guide/
│       ├── SKILL.md
│       └── rules/api-keys-setup.md
├── hooks/hooks.json                     Pre/Post 도구 사용 훅
├── scripts/
│   ├── setup-check.sh
│   ├── check-mcp-server.sh
│   └── post-tool-note.sh
├── LICENSE                              MIT
└── README.md                            이 파일
```

---

## 원본 대비 변경사항

| 변경 유형 | 파일 | 내용 |
|----------|------|------|
| 신규 | `config.yaml` | AI 모델 선택 설정 파일 (프리셋 3종 + 커스텀) |
| 신규 | `commands/select-models.md` | 대화형 모델 선택 커맨드 |
| 신규 | `skills/.../model-providers.md` | 18개 프로바이더 상세 가이드 |
| 수정 | `commands/add-voiceover.md` | edge-tts, kokoro, coqui-xtts 무료 옵션 추가 |
| 수정 | `commands/add-music.md` | MusicGen, ACE-Step 무료 옵션 추가 |
| 수정 | `commands/generate-image.md` | Pixazo, FLUX 로컬 무료 옵션 추가 |
| 수정 | `commands/generate-clip.md` | Google AI Studio, CogVideo 무료 옵션 추가 |
| 수정 | `commands/transcribe.md` | faster-whisper 무료 옵션 추가 |
| 수정 | `commands/create-video.md` | config.yaml 기반 프로바이더 분기 |
| 수정 | `commands/setup.md` | 모델 선택 단계 + 프리셋별 설치 |
| 수정 | `skills/.../SKILL.md` | 모델 프로바이더 시스템 섹션 추가 |

---

## 라이선스

MIT License — 원본과 동일. 자유롭게 사용, 수정, 배포 가능.

단, AI 생성물(음악, 영상, 이미지)의 상업적 사용은 각 AI 서비스의 이용약관을 별도 확인해야 합니다.

---

## 크레딧

- 원본: [DojoCodingLabs/remotion-superpowers](https://github.com/DojoCodingLabs/remotion-superpowers) by Juan C. Guerrero
- Fork 수정: AI 모델 선택 시스템 추가
- 영상 엔진: [Remotion](https://remotion.dev) — React 기반 프로그래밍 영상 제작

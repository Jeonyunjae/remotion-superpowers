# Remotion Superpowers (Custom Fork)

> AI 영상 제작 자동화 패키지 — Claude Code + Remotion 기반
> 원본: [DojoCodingLabs/remotion-superpowers](https://github.com/DojoCodingLabs/remotion-superpowers) (MIT)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

---

## 제작 의의

### 왜 이 패키지를 만들었는가?

기존 AI 영상 제작 도구들은 두 가지 문제가 있습니다:

1. **프리프로덕션의 부재** — 대부분의 도구가 "프롬프트 입력 → 영상 출력"의 단순 구조입니다. 컨셉, 전략, 스크립트, 스토리보드 등 영상의 품질을 결정하는 사전 기획 단계가 없어서, AI가 추측한 결과물이 나옵니다.

2. **질문 없이 진행** — 영상에 어떤 인물이 등장하는지, 어떤 복장인지, 어떤 카메라 앵글인지, 배경은 어떤 모습인지 사용자에게 물어보지 않습니다. 결과적으로 사용자의 비전이 아닌 AI의 추측으로 영상이 만들어집니다.

3. **납품 이후의 부재** — 렌더링이 끝나면 끝입니다. 다중 포맷 출력, 법적 검토, 접근성, SEO, 성과 측정 등 프로페셔널 영상 제작에 필수적인 후속 프로세스가 없습니다.

### 이 패키지의 해결 방식

**"묻고, 기록하고, 만든다 (Ask, Document, Produce)"**

- 업계 표준 영상 제작 프로세스(광고 에이전시, 기업 영상, 모션그래픽)를 분석하여 **6단계 파이프라인**(기획→프리프로덕션→프로덕션→포스트프로덕션→납품→공개 후)을 구현
- 매 단계마다 사용자에게 **구체적인 질문**을 하고, 모호한 답변에는 **선택지를 제시**하여 명확한 정보를 확보
- 모든 의사결정을 **문서로 기록**하고, 승인 게이트를 통과한 문서만이 다음 단계의 입력이 됨
- 기능별로 **무료/유료 AI 모델을 선택** 가능 (원본의 KIE 유료 API 종속에서 탈피)

### 기존 패키지와의 차별점

| 특징 | 원본 remotion-superpowers | 이 Fork |
|------|--------------------------|---------|
| 프리프로덕션 | 없음 | 6단계 (브리프~프롬프트 시트) + 8개 보강 커맨드 |
| 질문 프로토콜 | 없음 | 모호함 감지 + 선택지 제시 + 점진적 깊이 |
| 승인 게이트 | 없음 | Treatment, Script, Storyboard, Legal 4곳 |
| AI 모델 선택 | KIE 고정 (유료) | 18개 프로바이더, 3단계 프리셋 (무료/$0부터) |
| 포스트프로덕션 | AI 리뷰만 | QC, 접근성, 썸네일, SEO, 수정관리 |
| 납품 | 없음 | 다중 포맷, 납품 패키지, 다국어, 법적 검토 |
| 공개 후 | 없음 | 콘텐츠 재활용, 아카이빙 |
| 커맨드 수 | 13개 | **40개** |
| 업계 표준 커버율 | ~20% | **~85%** |

---

## 전체 프로세스 (6단계, 40개 커맨드)

### 프로세스 흐름도

> 범례: `[필수]` 반드시 실행 · `[선택]` 필요시 실행 · `★승인` 사용자 승인 후 다음 진행

```
=== Phase 1: 기획 (Planning) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[필수] /project-scope   프로젝트 범위·일정·예산·수정횟수를 질문하여 정의    → docs/00-project-scope.md
[선택] /brand-kit       기존 브랜드 가이드라인(로고·색상·폰트·톤) 입력     → docs/00-brand-guidelines.md
[필수] /select-models   AI 프로바이더를 기능별로 선택 (무료/유료 프리셋)    → config.yaml


=== Phase 2: 프리프로덕션 (Pre-production) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[필수] /receive-brief   클라이언트 요구사항을 10개 질문으로 체계적 수령      → docs/01-client-brief.md
[필수] /creative-brief  브리프 기반 커뮤니케이션 전략 수립 (5개 전략 질문)   → docs/02-creative-brief.md
[선택] /concept-options 2~3개 크리에이티브 컨셉을 비교·평가                → docs/03-concepts.md
[필수] /treatment       영상의 비주얼 방향·스타일·무드 제안 (6개 질문)      → docs/03-treatment.md ★승인
[필수] /write-script    장면별 나레이션·화면·음악 지시 포함 AV 스크립트 작성  → docs/04-script.md ★승인
[선택] /fact-check      스크립트 내 데이터·통계의 정확성 검증              → docs/04-fact-check.md
[필수] /storyboard      각 장면의 피사체·카메라·배경을 질문하여 확정 (씬당 8개) → docs/05-storyboard.md ★승인
[선택] /style-frame     고퀄리티 디자인 시안 이미지 3~5장 생성             → docs/style-frames/
[선택] /animatic        스토리보드를 Remotion으로 타이밍 프리뷰            → src/Animatic.tsx
[필수] /prompt-sheet    스토리보드를 AI 생성용 프롬프트·스타일·모델로 변환    → docs/06-*.md (3개 파일)
[필수] /legal-check     AI 생성물 라이선스·저작권·초상권 법적 검토          → docs/07-legal-checklist.md ★승인


=== Phase 3: 프로덕션 (Production) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[필수] /create-video    프리프로덕션 문서 기반 전체 영상 제작 파이프라인 실행  → out/video.mp4
[선택] /create-short    세로형 숏폼 영상 제작 (TikTok/Reels/Shorts)      → out/short.mp4
[필수] /add-voiceover   스크립트 텍스트로 TTS 나레이션 생성 + 컴포지션 연결  → public/audio/voiceover.mp3
[필수] /add-music       장면 무드에 맞는 배경음악 생성 + 페이드·더킹 적용   → public/audio/music.wav
[선택] /generate-image  프롬프트 시트 기반 장면별 AI 이미지 생성            → public/images/
[선택] /generate-clip   AI 영상 클립 생성 (텍스트→영상, 이미지→영상)       → public/footage/
[선택] /find-footage    Pexels에서 키워드로 스톡 영상/사진 검색·다운로드    → public/footage/
[선택] /add-captions    나레이션 음성을 분석하여 TikTok 스타일 자막 생성     → 자막 컴포넌트
[선택] /add-transitions 장면 간 전환 효과 추가 (fade, slide, wipe, flip)  → 전환 컴포넌트
[선택] /transcribe      오디오/비디오 파일을 텍스트로 변환 (SRT 자막 생성)   → SRT 파일
[선택] /audio-mix       플랫폼별 LUFS 표준 적용 + 음량 밸런스 최적화       → 오디오 최적화
[선택] /color-grade     장면 간 색감 일관성 검사 + CSS 필터로 보정          → 컬러 보정
[선택] /analyze-footage 기존 영상을 AI로 분석 (장면탐지·객체인식·요약)      → 분석 결과


=== Phase 4: 포스트프로덕션 (Post-production) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[필수] /review-video    렌더링된 영상을 AI가 4개 차원으로 리뷰·피드백       → 리뷰 피드백
[선택] /revision-log    수정 요청을 라운드별로 접수·추적·버전 관리          → docs/08-revision-log.md
[필수] /qc-check        ffprobe로 코덱·해상도·LUFS·파일크기 기술 검증      → QC 리포트
[선택] /accessibility   WCAG 2.1 AA 기준 자막·색상대비·깜빡임 검증        → 접근성 리포트
[선택] /thumbnail       CTR 최적화된 썸네일 생성 (A/B 시안 포함)           → out/thumbnails/
[선택] /seo-metadata    플랫폼별 제목·설명·태그·해시태그·챕터 생성          → docs/10-seo-metadata.md


=== Phase 5: 납품 (Delivery) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[필수] /export-multi    하나의 소스에서 플랫폼별 포맷으로 변환 출력         → 플랫폼별 MP4
[필수] /deliver         최종 파일+스펙시트+크레딧+사용가이드 납품 패키지 구성 → docs/09-delivery-package.md
[선택] /localize        자막 번역 + TTS 더빙으로 다국어 버전 생성          → 다국어 MP4 + SRT


=== Phase 6: 공개 후 (Post-release) ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[선택] /repurpose       완성 영상에서 클립·GIF·블로그·오디오 등 파생물 생성  → out/repurposed/
[선택] /archive         프로젝트 회고 + 에셋 출처 정리 + 아카이빙          → docs/99-archive.md


=== 유틸리티 ===

커맨드              역할                                         산출물
─────────────────────────────────────────────────────────────────────────────────
[아무때나] /setup       환경 점검 + 폴더 구조 생성 + AI 모델 설정          → 프로젝트 스캐폴드
[아무때나] /progress    산출물 폴더 스캔하여 진행 상황 자동 파악·갱신       → docs/00-progress.md
```

### 승인 체크포인트 (★ 표시)

| 체크포인트 | 커맨드 | 확인 내용 |
|-----------|--------|----------|
| 1차 승인 | `/treatment` | 비주얼 방향, 스타일, 색감, 음악 방향 |
| 2차 승인 | `/write-script` | 나레이션 대사, 장면 구성, 전체 흐름 |
| 3차 승인 | `/storyboard` | 장면별 구도, 인물, 카메라, 배경 |
| 4차 승인 | `/legal-check` | AI 생성물 라이선스, 저작권, 크레딧 |

승인 없이 다음 단계로 진행할 수 없습니다. 수정 요청 시 해당 단계만 수정 후 재승인합니다.

---

## 산출물 (문서 + 에셋 + 최종 파일)

### 프리프로덕션 산출물 (14개 문서)

| # | 파일 | 커맨드 | 내용 |
|---|------|--------|------|
| 1 | `docs/00-project-scope.md` | `/project-scope` | 프로젝트 범위, 딜리버러블, 수정횟수 제한, 마일스톤 일정, 예산, 소유권 |
| 2 | `docs/00-brand-guidelines.md` | `/brand-kit` | 로고 파일+배치 규칙, 색상 팔레트(HEX), 폰트, 톤 오브 보이스, 금지 요소, 브랜드 체크리스트 |
| 3 | `docs/01-client-brief.md` | `/receive-brief` | 영상 목적, 타깃 시청자, 톤앤매너, 길이, 배포 채널, 예산, 일정, 필수 포함 요소, 제약 사항, 기존 소스 |
| 4 | `docs/02-creative-brief.md` | `/creative-brief` | 커뮤니케이션 목표(인지/감정/행동), 타깃 프로파일, 핵심 메시지, 톤앤매너 상세, 경쟁 환경, KPI |
| 5 | `docs/03-concepts.md` | `/concept-options` | 2~3개 크리에이티브 컨셉 비교표 (접근 방식, 비주얼, 톤, 제작 난이도, 비용 비교) |
| 6 | `docs/03-treatment.md` | `/treatment` | 내러티브 구조, 비주얼 스타일, 색감/조명, 무드 레퍼런스, 음악/사운드, 캐릭터, 편집 스타일, AI 도구 계획 |
| 7 | `docs/04-script.md` | `/write-script` | AV 포맷 장면별 스크립트 (화면 설명 + 나레이션 전문 + 자막 + 음악 지시 + 효과음), 전체 나레이션 연결 텍스트 |
| 8 | `docs/04-fact-check.md` | `/fact-check` | 수치/통계/인용 출처 확인, 검증 결과, 수정 권고사항 |
| 9 | `docs/05-storyboard.md` | `/storyboard` | 샷별 피사체/인물/동선/카메라/배경/전환/오디오/효과, 프리뷰 이미지, 전체 흐름 요약표 |
| 10 | `docs/06-prompt-sheet.md` | `/prompt-sheet` | 샷별 AI 생성 프롬프트 (Positive/Negative), 카메라/스타일 키워드, 해상도, 모션 강도, 시드 |
| 11 | `docs/06-style-guide.md` | `/prompt-sheet` | 공통 프롬프트 프리픽스, 색감/톤 키워드, 캐릭터 일관성 규칙, 글로벌 네거티브, 일관성 체크리스트 |
| 12 | `docs/06-model-selection.md` | `/prompt-sheet` | 샷별 모델 배정표, 비용 예측 합계, 선택 근거 |
| 13 | `docs/07-legal-checklist.md` | `/legal-check` | 프로바이더별 라이선스 검증, 크레딧 표시 필요 목록, 리스크 항목, 최종 판정 |
| 14 | `docs/style-frames/` | `/style-frame` | 핵심 장면 3~5장의 고퀄리티 디자인 시안 이미지 |

### 프로덕션 에셋

| 유형 | 경로 | 생성 커맨드 |
|------|------|-----------|
| 나레이션 | `public/audio/voiceover.mp3` | `/add-voiceover` |
| 배경음악 | `public/audio/music.wav` | `/add-music` |
| 효과음 | `public/audio/sfx/` | `/create-video` |
| AI 이미지 | `public/images/` | `/generate-image` |
| AI 영상 클립 | `public/footage/` | `/generate-clip` |
| 스톡 영상 | `public/footage/` | `/find-footage` |
| 브랜드 에셋 | `public/brand/` | `/brand-kit` |

### 포스트프로덕션 산출물

| # | 파일 | 커맨드 | 내용 |
|---|------|--------|------|
| 1 | `docs/08-revision-log.md` | `/revision-log` | 수정 라운드별 피드백, 타임코드별 수정 내역, 버전 이력 |
| 2 | `docs/10-seo-metadata.md` | `/seo-metadata` | YouTube/Instagram/TikTok/LinkedIn별 제목, 설명, 태그, 해시태그, 챕터 마커 |
| 3 | `out/thumbnails/` | `/thumbnail` | 클릭 최적화 썸네일 (1280x720, 2MB 이하) |
| 4 | QC 리포트 | `/qc-check` | 코덱, 해상도, FPS, LUFS, 파일 크기 검증 결과 |
| 5 | 접근성 리포트 | `/accessibility` | 자막, 색상 대비, 깜빡임, 오디오 디스크립션 검증 |

### 최종 납품 산출물

| # | 파일 | 커맨드 | 내용 |
|---|------|--------|------|
| 1 | `out/video.mp4` | `/create-video` | 마스터 영상 (16:9) |
| 2 | `out/video-vertical.mp4` | `/export-multi` | 세로 버전 (9:16) |
| 3 | `out/video-square.mp4` | `/export-multi` | 정방형 버전 (1:1) |
| 4 | `out/preview.gif` | `/export-multi` | GIF 프리뷰 |
| 5 | `delivery/` | `/deliver` | 전체 납품 폴더 (영상+썸네일+자막+문서) |
| 6 | `docs/09-delivery-package.md` | `/deliver` | 스펙시트, 에셋 크레딧, 사용 가이드라인 |
| 7 | 다국어 버전 | `/localize` | 언어별 MP4 + SRT |
| 8 | `docs/99-archive.md` | `/archive` | 프로젝트 회고, 에셋 출처 목록, 전체 구조 |

---

## 사용 방법

### 1. 설치

#### 필수 요건
- Claude Code (플러그인 지원 버전)
- Node.js >= 18
- Python 3 + pip (무료 프로바이더 사용 시)

#### 설치 순서

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

#### 무료 프리셋 빠른 설치

```bash
pip install edge-tts faster-whisper transformers scipy soundfile torch
```

### 2. 프로젝트 시작 (처음부터 끝까지)

#### Step 1: 기획

```
/project-scope    ← 범위, 딜리버러블, 일정, 예산 정의
/brand-kit        ← 로고, 색상, 폰트, 톤 가이드 입력
/select-models    ← AI 프로바이더 선택 (free/budget/premium)
```

#### Step 2: 프리프로덕션

```
/receive-brief    ← 영상 컨셉/요청 전달 → 10개 항목 질문 → 브리프 문서
/creative-brief   ← 전략 질문 5개 → 크리에이티브 브리프
/concept-options  ← [선택] 2~3개 컨셉 비교
/treatment        ← 비주얼 질문 6개 → 트리트먼트 → ★승인
/write-script     ← 스크립트 질문 5개 → AV 스크립트 → ★승인
/fact-check       ← [선택] 데이터 정확성 검증
/storyboard       ← 장면별 8개 질문 → 스토리보드 + 프리뷰 → ★승인
/style-frame      ← [선택] 고퀄 디자인 시안 3~5장
/animatic         ← [선택] 타이밍 프리뷰 영상
/prompt-sheet     ← AI 프롬프트 시트 + 스타일 가이드 + 모델 선택
/legal-check      ← 법적 검토 → ★승인
```

#### Step 3: 프로덕션

```
/create-video     ← 전체 제작 파이프라인 실행
                     Phase 1: 컨셉 브레이크다운
                     Phase 2: 오디오 에셋 생성 (나레이션 → 음악 → 효과음)
                     Phase 3: 비주얼 에셋 확보 (스톡 → 이미지 → 영상 클립)
                     Phase 4: Remotion 코드 생성 (장면 컴포넌트 + 시퀀싱)
                     Phase 5: 프리뷰 및 수정
                     Phase 6: 렌더링 → MP4

/audio-mix        ← LUFS 표준 적용, 나레이션/BGM 밸런스
/color-grade      ← AI 이미지 간 색감 일관성 보정
```

#### Step 4: 포스트프로덕션

```
/review-video     ← AI가 영상을 분석하고 피드백
/revision-log     ← 수정 피드백 접수 → 수정 → 재렌더링
/qc-check         ← 기술 사양 자동 검증 (코덱/LUFS/해상도)
/accessibility    ← [선택] 접근성 검증 (WCAG 2.1 AA)
/thumbnail        ← 썸네일 생성 (CTR 최적화)
/seo-metadata     ← 제목/설명/태그/해시태그/챕터마커
```

#### Step 5: 납품

```
/export-multi     ← 플랫폼별 버전 생성 (가로/세로/정방형/GIF)
/deliver          ← 납품 패키지 구성 (파일+스펙시트+크레딧+가이드)
/localize         ← [선택] 다국어 버전 (자막 번역/TTS 더빙)
```

#### Step 6: 공개 후

```
/repurpose        ← 콘텐츠 재활용 (하이라이트/숏폼/GIF/블로그/오디오)
/archive          ← 프로젝트 아카이빙 + 회고
```

### 3. 질문 프로토콜 (Questioning Protocol)

이 패키지의 핵심 원칙은 **"추측하지 말고 질문하라"**입니다.

#### 모호함 감지

사용자의 답변에 다음이 포함되면 구체화 질문을 합니다:
- "적당히", "알아서", "그냥", "대충", "아무거나", "상관없어"
- "좋은 느낌으로", "예쁘게", "멋지게"
- 5단어 미만의 짧은 답변

#### 대응 방식

```
사용자: "분위기는 적당히 좋게 해줘"
에이전트: "좀 더 구체적으로 정해볼게요. 다음 중 어떤 느낌이 가까운가요?
  1. 밝고 에너지 넘치는 (광고, 홍보)
  2. 차분하고 신뢰감 있는 (기업, 교육)
  3. 긴장감 있고 시네마틱한 (다큐멘터리, 안전)
  4. 따뜻하고 감성적인 (브랜딩, 스토리)"
```

#### "알아서 해줘" 대응

사용자가 "알아서 해줘"라고 하면:
1. 구체적인 제안을 2~3가지 제시
2. 각 제안의 차이점 설명
3. 하나를 선택하게 함
4. 선택 없이 진행하지 않음

---

## AI 모델 선택 시스템

### 비용 프리셋 3종

| 프리셋 | 비용 | TTS | 음악 | 이미지 | 영상 | 자막 | SFX |
|--------|------|-----|------|--------|------|------|-----|
| **free** | **$0** | edge-tts | MusicGen | Pixazo | Google AI Studio | faster-whisper | Freesound |
| **budget** | ~$1~5/영상 | edge-tts | MusicGen | Replicate | Replicate | faster-whisper | Freesound |
| **premium** | KIE 요금제 | ElevenLabs | Suno | KIE | Veo 3.1 | KIE Whisper | ElevenLabs |

### 프리셋 전환

```bash
/select-models          # 대화형 모델 선택
# 또는 config.yaml 직접 수정
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

| 프로바이더 | 비용 | 클립 길이 | 설치 | 특징 |
|-----------|------|----------|------|------|
| google-ai-studio | 무료 | 3~8초 | API 키 (무료) | Veo 3.1, 속도 제한 |
| cogvideo-local | 무료 | 3~6초 | GPU 16GB+ | CogVideoX-2B |
| replicate | ~$0.05/클립 | 3~10초 | API 키만 | Wan 2.5, Kling 2.6 |
| kie | 유료 | 3~8초 | API 키만 | Veo 3.1 통합 |

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

## 커맨드 전체 목록 (40개)

### Phase 1: 기획 (3개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/project-scope` | 프로젝트 범위/일정/예산/수정횟수 정의 | `docs/00-project-scope.md` |
| `/brand-kit` | 브랜드 가이드라인 입력 (로고/색상/폰트/톤) | `docs/00-brand-guidelines.md` |
| `/select-models` | AI 프로바이더 선택 (대화형) | `config.yaml` |

### Phase 2: 프리프로덕션 (11개)

| 커맨드 | 설명 | 산출물 | 승인 |
|--------|------|--------|------|
| `/receive-brief` | 클라이언트 브리프 수령 (10개 필수 질문) | `docs/01-client-brief.md` | - |
| `/creative-brief` | 내부 크리에이티브 전략 (5개 전략 질문) | `docs/02-creative-brief.md` | - |
| `/concept-options` | 복수 컨셉 비교 (2~3안) | `docs/03-concepts.md` | 선택 |
| `/treatment` | 비주얼 방향 제안 (6개 비주얼 질문) | `docs/03-treatment.md` | **필수** |
| `/write-script` | AV 스크립트 작성 (5개 방향 질문) | `docs/04-script.md` | **필수** |
| `/fact-check` | 데이터/통계 정확성 검증 | `docs/04-fact-check.md` | 선택 |
| `/storyboard` | 장면별 Q&A + AI 프리뷰 (씬당 8개 질문) | `docs/05-storyboard.md` | **필수** |
| `/style-frame` | 고퀄리티 디자인 시안 (3~5장) | `docs/style-frames/` | 선택 |
| `/animatic` | 타이밍 프리뷰 (움직이는 스토리보드) | `src/Animatic.tsx` | 선택 |
| `/prompt-sheet` | AI 생성 프롬프트 + 스타일 가이드 + 모델 선택 | `docs/06-*.md` (3개) | - |
| `/legal-check` | 법적 체크리스트 (라이선스/저작권/초상권) | `docs/07-legal-checklist.md` | **필수** |

### Phase 3: 프로덕션 (13개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/create-video` | 전체 영상 제작 파이프라인 (6단계) | `out/video.mp4` |
| `/create-short` | 숏폼 세로 영상 (TikTok/Reels/Shorts) | `out/short.mp4` |
| `/add-voiceover` | 나레이션 생성 + 컴포지션 연결 | `public/audio/voiceover.mp3` |
| `/add-music` | 배경음악 생성 + 페이드인/아웃 + 더킹 | `public/audio/music.wav` |
| `/generate-image` | AI 이미지 생성 | `public/images/` |
| `/generate-clip` | AI 영상 클립 생성 (T2V, I2V) | `public/footage/` |
| `/find-footage` | Pexels 스톡 영상/사진 검색 + 다운로드 | `public/footage/` |
| `/add-captions` | TikTok 스타일 애니메이션 자막 | 자막 컴포넌트 |
| `/add-transitions` | 장면 전환 효과 (fade, slide, wipe, flip) | 전환 컴포넌트 |
| `/transcribe` | 오디오/비디오 → 텍스트 변환 (SRT) | SRT 파일 |
| `/audio-mix` | 오디오 믹싱 (LUFS 표준, 더킹, 밸런스) | 최적화된 오디오 |
| `/color-grade` | 컬러 일관성 검사 + CSS 필터 보정 | 컬러 보정 |
| `/analyze-footage` | 기존 영상 AI 분석 (장면탐지, 객체인식) | 분석 결과 |

### Phase 4: 포스트프로덕션 (6개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/review-video` | AI 리뷰 (비주얼/페이싱/싱크/효과) | 리뷰 피드백 |
| `/revision-log` | 수정 라운드 관리 (피드백 접수/추적/버전관리) | `docs/08-revision-log.md` |
| `/qc-check` | 기술 QC (ffprobe 기반 코덱/LUFS/해상도 검증) | QC 리포트 |
| `/accessibility` | 접근성 검증 (WCAG 2.1 AA, 자막, 색상대비, 깜빡임) | 접근성 리포트 |
| `/thumbnail` | 썸네일 생성 (CTR 최적화, A/B 시안) | `out/thumbnails/` |
| `/seo-metadata` | SEO 메타데이터 (제목/설명/태그/해시태그/챕터) | `docs/10-seo-metadata.md` |

### Phase 5: 납품 (3개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/export-multi` | 다중 포맷 출력 (16:9, 9:16, 1:1, GIF) | 플랫폼별 MP4 |
| `/deliver` | 최종 납품 패키지 (파일+스펙시트+크레딧+가이드) | `docs/09-delivery-package.md` + `delivery/` |
| `/localize` | 다국어 버전 (자막 번역 + TTS 더빙) | 언어별 MP4 + SRT |

### Phase 6: 공개 후 (2개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/repurpose` | 콘텐츠 재활용 (클립/GIF/블로그/오디오 추출) | `out/repurposed/` |
| `/archive` | 프로젝트 아카이빙 + 회고 | `docs/99-archive.md` |

### 유틸리티 (2개)

| 커맨드 | 설명 | 산출물 |
|--------|------|--------|
| `/setup` | 프로젝트 초기 설정 (환경 + 폴더 구조 + AI 모델) | 프로젝트 스캐폴드 + `docs/00-progress.md` |
| `/progress` | 진행 상황 대시보드 (산출물 스캔 → 자동 갱신) | 터미널 출력 + `docs/00-progress.md` 갱신 |

---

## 프로젝트 구조

### 패키지 구조 (설치되는 플러그인)

```
remotion-superpowers/                    ← 플러그인 패키지
│
├── .claude-plugin/                      플러그인 메타
│   ├── plugin.json
│   └── marketplace.json
├── .mcp.json                            5개 MCP 서버 설정
├── config.yaml                          AI 모델 선택 설정
│
├── commands/                            40개 슬래시 커맨드
│   ├── [Phase 1] project-scope, brand-kit, select-models
│   ├── [Phase 2] receive-brief, creative-brief, concept-options,
│   │             treatment, write-script, fact-check, storyboard,
│   │             style-frame, animatic, prompt-sheet, legal-check
│   ├── [Phase 3] create-video, create-short, add-voiceover, add-music,
│   │             generate-image, generate-clip, find-footage,
│   │             add-captions, add-transitions, transcribe,
│   │             audio-mix, color-grade, analyze-footage
│   ├── [Phase 4] review-video, revision-log, qc-check,
│   │             accessibility, thumbnail, seo-metadata
│   ├── [Phase 5] export-multi, deliver, localize
│   ├── [Phase 6] repurpose, archive
│   └── [유틸리티] setup, progress
│
├── agents/                              3개 AI 에이전트
│   ├── video-director.md
│   ├── media-scout.md
│   └── post-producer.md
│
├── skills/
│   └── remotion-production/
│       ├── SKILL.md                     메인 스킬 정의
│       └── rules/                       21개 규칙 파일
│           ├── questioning-protocol.md  질문 프로토콜 (공통)
│           ├── progress-tracking.md     진행 상황 자동 갱신 (공통)
│           ├── production-pipeline.md   E2E 파이프라인
│           ├── model-providers.md       프로바이더 가이드
│           └── ... (17개 추가 규칙)
│
├── hooks/hooks.json
└── README.md
```

### 산출물 폴더 구조 (`/setup` 실행 시 프로젝트에 생성)

```
your-remotion-project/
│
├── docs/                                📄 기획·프리프로덕션 산출물
│   ├── 00-progress.md                   진행 상황 대시보드 (자동 갱신)
│   ├── 00-project-scope.md              ← /project-scope
│   ├── 00-brand-guidelines.md           ← /brand-kit
│   ├── 01-client-brief.md               ← /receive-brief
│   ├── 02-creative-brief.md             ← /creative-brief
│   ├── 03-concepts.md                   ← /concept-options
│   ├── 03-treatment.md                  ← /treatment ★승인
│   ├── 04-script.md                     ← /write-script ★승인
│   ├── 04-fact-check.md                 ← /fact-check
│   ├── 05-storyboard.md                 ← /storyboard ★승인
│   ├── 06-prompt-sheet.md               ← /prompt-sheet
│   ├── 07-legal-checklist.md            ← /legal-check ★승인
│   ├── 08-revision-log.md               ← /revision-log
│   ├── 09-delivery-package.md           ← /deliver
│   ├── 10-seo-metadata.md               ← /seo-metadata
│   ├── 99-archive.md                    ← /archive
│   ├── storyboard-previews/             ← /storyboard (프리뷰 이미지)
│   └── style-frames/                    ← /style-frame (디자인 시안)
│
├── public/                              🎵 미디어 에셋
│   ├── audio/
│   │   ├── voiceover.mp3                ← /add-voiceover
│   │   └── music.wav                    ← /add-music
│   ├── images/                          ← /generate-image
│   └── footage/                         ← /generate-clip, /find-footage
│
├── out/                                 🎬 렌더링 결과물
│   ├── video.mp4                        ← /create-video
│   ├── short.mp4                        ← /create-short
│   ├── thumbnails/                      ← /thumbnail
│   └── repurposed/                      ← /repurpose
│
├── delivery/                            📦 납품 패키지
│   └── (최종 파일 + 스펙시트)            ← /deliver
│
└── config.yaml                          ← /select-models
```

> 세션을 다시 시작해도 `/progress`를 실행하면 이 폴더를 스캔하여 어디까지 진행했는지 자동으로 파악합니다.

---

## 라이선스

MIT License — 원본과 동일. 자유롭게 사용, 수정, 배포 가능.

단, AI 생성물(음악, 영상, 이미지)의 상업적 사용은 각 AI 서비스의 이용약관을 별도 확인해야 합니다. `/legal-check` 커맨드로 프로바이더별 라이선스를 검증할 수 있습니다.

---

## 크레딧

- 원본: [DojoCodingLabs/remotion-superpowers](https://github.com/DojoCodingLabs/remotion-superpowers) by Juan C. Guerrero
- Fork 수정: 6단계 프로덕션 파이프라인, AI 모델 선택 시스템, 질문 프로토콜
- 영상 엔진: [Remotion](https://remotion.dev) — React 기반 프로그래밍 영상 제작

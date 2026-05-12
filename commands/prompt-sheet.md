---
name: prompt-sheet
description: Convert the approved storyboard into AI generation prompts. Create a prompt sheet for each shot, an AI style guide for visual consistency, and a model selection document.
---

# Prompt Sheet — AI 생성 프롬프트 시트

승인된 스토리보드를 AI 이미지/영상 생성 프롬프트로 변환합니다. 샷별 프롬프트 시트, 비주얼 일관성을 위한 AI 스타일 가이드, 모델 선택 문서를 작성합니다.

## 사전 조건

다음 파일이 모두 존재해야 합니다:
- `docs/01-client-brief.md`
- `docs/02-creative-brief.md`
- `docs/03-treatment.md`
- `docs/04-script.md`
- `docs/05-storyboard.md`

없으면 해당 커맨드를 먼저 실행하도록 안내합니다.

## 프로세스

### Step 1: 이전 문서 읽기

```bash
cat docs/05-storyboard.md
cat docs/03-treatment.md
cat docs/01-client-brief.md
```

`config.yaml`도 읽어서 사용 가능한 프로바이더를 확인합니다:

```bash
cat config.yaml 2>/dev/null
```

### Step 2: 프롬프트 시트 작성

**출력 파일**: `docs/06-prompt-sheet.md`

```markdown
# Prompt Sheet

## 공통 설정
- 기본 해상도: [1920x1080 / 1080x1920]
- 기본 스타일: [스타일 키워드]
- 공통 프리픽스: [모든 프롬프트에 붙는 공통 지시어]
- 공통 네거티브: [모든 프롬프트에 적용하는 네거티브 프롬프트]

---

### Shot 1
- **생성 유형**: [이미지 / 영상 / 스톡 영상]
- **사용 모델**: [프로바이더:모델명]

**Positive Prompt:**
> [상세한 생성 프롬프트 — 피사체, 환경, 스타일, 카메라, 조명을 모두 포함]

**Negative Prompt:**
> [금지 요소 — ugly, blurry, deformed, watermark, text 등]

**스타일 키워드:**
[cinematic, photorealistic, illustration, 3d render 등]

**카메라 키워드:**
[medium shot, eye level, tracking shot, shallow depth of field 등]

**참조 이미지:**
[있을 경우 경로 — storyboard-previews/shot-01.png 등]

**생성 파라미터:**
- 해상도: [width x height]
- 모션 강도 (영상인 경우): [0.0~1.0]
- 시드: [고정 시드가 필요하면 기재]
- 기타: [cfg_scale, steps 등]

---

### Shot 2
[위와 동일한 형식으로 반복]

---

[모든 샷에 대해 반복]
```

### Step 3: AI 스타일 가이드 작성

**출력 파일**: `docs/06-style-guide.md`

```markdown
# AI Style Guide

## 1. 공통 프롬프트 프리픽스
[모든 이미지/영상 생성 프롬프트 앞에 붙일 공통 지시어]

예시: "cinematic lighting, professional color grading, 8k quality, detailed"

## 2. 색감/톤 키워드 세트
- 주요 색감: [키워드 목록]
- 조명 스타일: [키워드 목록]
- 분위기: [키워드 목록]

## 3. 캐릭터 일관성 규칙
[영상 전체에서 동일 인물/캐릭터가 일관되게 보이기 위한 규칙]

- 인물 묘사 고정 프롬프트: [구체적인 외모/복장 묘사 — 모든 샷에서 동일하게 사용]
- 참조 이미지 활용: [Image-to-Image 또는 ControlNet 사용 여부]
- 시드 고정: [동일 시드 사용 여부]

## 4. 금지 요소 (Global Negative)
[AI 생성물에서 반드시 피해야 할 아티팩트]

- 기본: ugly, blurry, deformed, disfigured, watermark, signature, text, logo
- 인물: extra limbs, extra fingers, missing fingers, bad anatomy, bad proportions
- 추가: [프로젝트 특수 금지 요소]

## 5. 일관성 체크리스트
- [ ] 모든 샷에 공통 프리픽스 적용
- [ ] 동일 인물의 묘사가 샷 간에 일치
- [ ] 배경 조명/시간대가 스토리보드와 일치
- [ ] 색감이 트리트먼트의 팔레트와 일치
```

### Step 4: 모델 선택 문서 작성

**출력 파일**: `docs/06-model-selection.md`

```markdown
# Model Selection

## config.yaml 프리셋
현재 프리셋: [free / budget / premium / custom]

## 샷별 모델 배정

| Shot | 생성 유형 | 프로바이더 | 모델 | 예상 비용 |
|------|----------|-----------|------|----------|
| 1 | 이미지 | [provider] | [model] | [cost] |
| 2 | 영상 클립 | [provider] | [model] | [cost] |
| 3 | 스톡 영상 | Pexels | - | 무료 |
| ... | ... | ... | ... | ... |

## 비용 예측 합계

| 항목 | 수량 | 단가 | 소계 |
|------|------|------|------|
| AI 이미지 생성 | [n]장 | [단가] | [소계] |
| AI 영상 클립 생성 | [n]개 | [단가] | [소계] |
| TTS 나레이션 | [n]글자 | [단가] | [소계] |
| 배경음악 | [n]곡 | [단가] | [소계] |
| 효과음 | [n]개 | [단가] | [소계] |
| **총 예상 비용** | | | **[합계]** |

## 모델 선택 근거
[각 모델을 선택한 이유 — 비용, 품질, 속도, 스타일 적합성]
```

### Step 5: 사용자 확인

> "프롬프트 시트, 스타일 가이드, 모델 선택 문서를 작성했습니다. 프리프로덕션이 완료되었습니다. 이제 `/create-video`로 실제 제작을 시작할 수 있습니다."

## 다음 단계

프리프로덕션 완료. `/create-video`를 실행하면 프롬프트 시트를 기반으로 에셋을 생성하고 영상을 제작합니다.

---
name: concept-options
description: Develop 2-3 creative concept options with visual references. Present side-by-side comparison for the user to choose or combine. Replaces single-direction treatment approach.
---

# Concept Options — 복수 컨셉 시안 비교

하나의 방향만 제시하는 대신, 2~3개의 서로 다른 크리에이티브 컨셉을 개발하여 비교합니다.

이 커맨드는 `/treatment` 전에 실행할 수 있는 선택적(optional) 단계입니다.

## 사전 조건

다음 파일이 존재해야 합니다:
- `docs/01-client-brief.md`
- `docs/02-creative-brief.md`

없으면 해당 커맨드를 먼저 실행하도록 안내합니다.

## 프로세스

### Step 1: 이전 문서 읽기

```bash
cat docs/01-client-brief.md
cat docs/02-creative-brief.md
cat docs/00-brand-guidelines.md 2>/dev/null
```

### Step 2: 컨셉 방향 질문

`rules/questioning-protocol.md`의 모호함 감지 규칙을 적용합니다.

> "컨셉 개발 전에 몇 가지 방향을 잡겠습니다."

| # | 질문 |
|---|------|
| 1 | "선호하는 영상 스타일이 있나요? (레퍼런스 영상 URL이나 설명)" |
| 2 | "절대 원하지 않는 방향이 있나요?" |
| 3 | "시청자를 움직이는 가장 큰 동기는 무엇인가요? (두려움, 호기심, 이익, 감동 등)" |

### Step 3: 3가지 컨셉 개발

각 컨셉은 명확히 다른 접근 방식이어야 합니다.

```bash
mkdir -p docs
```

**출력 파일**: `docs/03-concepts.md`

```markdown
# Concept Options

## 컨셉 A: [컨셉명]
- **접근 방식**: [한 줄 설명]
- **내러티브 구조**: [어떤 이야기 방식?]
- **비주얼 스타일**: [어떤 비주얼?]
- **톤앤매너**: [어떤 분위기?]
- **음악 방향**: [어떤 음악?]
- **강점**: [이 컨셉의 장점]
- **약점**: [이 컨셉의 단점/리스크]
- **적합한 경우**: [어떤 상황에 최적?]

## 컨셉 B: [컨셉명]
[위와 동일 형식]

## 컨셉 C: [컨셉명]
[위와 동일 형식]

## 비교 요약

| 항목 | 컨셉 A | 컨셉 B | 컨셉 C |
|------|--------|--------|--------|
| 접근 방식 | [요약] | [요약] | [요약] |
| 비주얼 | [요약] | [요약] | [요약] |
| 톤 | [요약] | [요약] | [요약] |
| 제작 난이도 | [상/중/하] | [상/중/하] | [상/중/하] |
| 예상 비용 | [금액] | [금액] | [금액] |
| 추천 이유 | [설명] | [설명] | [설명] |
```

### Step 4: AI 프리뷰 이미지 생성

각 컨셉의 대표 이미지를 1장씩 AI로 생성하여 시각적으로 비교할 수 있게 합니다.

```bash
mkdir -p docs/concept-previews
```

`config.yaml`의 이미지 프로바이더를 사용합니다.

### Step 5: 사용자 선택

> "3가지 컨셉을 준비했습니다. 각 컨셉의 프리뷰 이미지와 비교표를 확인해주세요.\n어떤 컨셉이 마음에 드나요?\n1. 컨셉 A 선택\n2. 컨셉 B 선택\n3. 컨셉 C 선택\n4. A와 B 조합 (구체적으로 어떤 요소?)\n5. 다른 방향 제안"

선택된 컨셉을 기반으로 `/treatment`를 진행합니다.

## 다음 단계

선택 완료 후 `/treatment`를 실행합니다. 선택된 컨셉 정보가 트리트먼트에 반영됩니다.

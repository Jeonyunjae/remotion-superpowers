---
name: archive
description: Archive a completed video project — organize all source files, document asset sources, write a project retrospective, and create a self-contained archive for future reference.
---

# Archive — 프로젝트 아카이빙

완료된 프로젝트의 모든 소스 파일, 문서, 에셋을 정리하고 프로젝트 회고를 작성합니다. 향후 수정, 재사용, 시리즈 후속편에 대비합니다.

## 프로세스

### Step 1: 프로젝트 현황 점검

```bash
# 문서 목록
ls docs/

# 에셋 목록
ls -la public/audio/ public/images/ public/footage/ public/brand/ 2>/dev/null

# 출력 파일
ls -la out/ delivery/ 2>/dev/null

# Git 상태
git log --oneline -10
```

### Step 2: 에셋 출처 목록 작성

프로젝트에 사용된 모든 에셋의 출처를 정리합니다:

```bash
cat docs/07-legal-checklist.md 2>/dev/null
cat docs/06-model-selection.md 2>/dev/null
cat config.yaml
```

### Step 3: 프로젝트 회고 질문

> "프로젝트를 마무리하며 몇 가지 질문드립니다:"

| # | 질문 |
|---|------|
| 1 | "잘된 점은 무엇이었나요?" |
| 2 | "다음에 개선할 점은?" |
| 3 | "예상과 달랐던 부분은?" |
| 4 | "이 프로젝트에서 배운 것은?" |
| 5 | "향후 이 영상의 수정/업데이트가 예상되나요?" |

### Step 4: 아카이브 문서 작성

**출력 파일**: `docs/99-archive.md`

```markdown
# Project Archive

## 프로젝트 정보
- 프로젝트명: [명칭]
- 기간: [시작일] ~ [종료일]
- 최종 산출물: [파일 목록]

## 프로젝트 구조

```
[project-root]/
├── docs/                    # 프리프로덕션 문서
│   ├── 00-project-scope.md
│   ├── 00-brand-guidelines.md
│   ├── 01-client-brief.md
│   ├── 02-creative-brief.md
│   ├── 03-treatment.md
│   ├── 04-script.md
│   ├── 04-fact-check.md
│   ├── 05-storyboard.md
│   ├── 06-prompt-sheet.md
│   ├── 06-style-guide.md
│   ├── 06-model-selection.md
│   ├── 07-legal-checklist.md
│   ├── 08-revision-log.md
│   ├── 09-delivery-package.md
│   ├── 10-seo-metadata.md
│   └── 99-archive.md
├── public/                  # 에셋
│   ├── audio/
│   ├── images/
│   ├── footage/
│   └── brand/
├── src/                     # Remotion 소스
├── out/                     # 렌더링 출력
├── delivery/                # 납품 패키지
└── config.yaml              # 모델 설정
```

## 에셋 출처 목록

| # | 에셋 | 유형 | 프로바이더 | 라이선스 | 파일 경로 |
|---|------|------|-----------|---------|----------|
| 1 | [에셋명] | [TTS/음악/이미지/영상/SFX] | [provider] | [license] | [path] |

## 사용 AI 모델/프로바이더

| 기능 | 프로바이더 | 모델 | 비용 |
|------|-----------|------|------|
| TTS | [provider] | [model] | [cost] |
| 음악 | [provider] | [model] | [cost] |
| 이미지 | [provider] | [model] | [cost] |
| 영상 | [provider] | [model] | [cost] |
| 자막 | [provider] | [model] | [cost] |
| SFX | [provider] | [model] | [cost] |
| **합계** | | | **[total]** |

## 수정 이력 요약
- 총 수정 라운드: [n]회
- 주요 수정 내용: [요약]

## 프로젝트 회고

### 잘된 점
- [항목 1]
- [항목 2]

### 개선할 점
- [항목 1]
- [항목 2]

### 교훈
- [항목 1]
- [항목 2]

## 향후 계획
- 수정/업데이트 예상: [있음/없음]
- 시리즈 후속편 계획: [있음/없음]
- 재활용 계획: [설명]
```

### Step 5: Git 커밋 (선택)

> "프로젝트 아카이브를 생성했습니다. Git에 커밋할까요?"

## 프로젝트 완료

아카이빙이 완료되면 프로젝트가 공식적으로 종료됩니다.

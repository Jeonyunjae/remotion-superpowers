---
name: progress
description: Scan project folders to check production progress. Updates docs/00-progress.md with current completion status. Works by detecting which deliverables exist in the folder structure created by /setup.
---

# Progress — 진행 상황 대시보드

`/setup`에서 생성된 폴더 구조(`docs/`, `public/`, `out/`, `delivery/`)를 스캔하여 각 단계별 완료 상태를 확인하고, `docs/00-progress.md`를 갱신합니다.

## 사전 조건

`docs/00-progress.md`가 존재해야 합니다. 없으면:
> "프로젝트가 초기화되지 않았습니다. 먼저 `/setup`을 실행해주세요."

## 프로세스

### Step 1: 산출물 존재 여부 확인

다음 파일/디렉토리의 존재 여부를 `ls` 또는 `test -f`로 확인합니다:

```bash
# Phase 1
test -f docs/00-project-scope.md    && echo "✅" || echo "⬜"   # 필수
test -f docs/00-brand-guidelines.md && echo "✅" || echo "⬜"   # 선택
test -f config.yaml                 && echo "✅" || echo "⬜"   # 필수

# Phase 2
test -f docs/01-client-brief.md     && echo "✅" || echo "⬜"   # 필수
test -f docs/02-creative-brief.md   && echo "✅" || echo "⬜"   # 필수
test -f docs/03-concepts.md         && echo "✅" || echo "⬜"   # 선택
test -f docs/03-treatment.md        && echo "✅" || echo "⬜"   # 필수 ★승인
test -f docs/04-script.md           && echo "✅" || echo "⬜"   # 필수 ★승인
test -f docs/04-fact-check.md       && echo "✅" || echo "⬜"   # 선택
test -f docs/05-storyboard.md       && echo "✅" || echo "⬜"   # 필수 ★승인
test -d docs/style-frames           && echo "✅" || echo "⬜"   # 선택 (파일 존재 시)
test -f src/Animatic.tsx            && echo "✅" || echo "⬜"   # 선택
test -f docs/06-prompt-sheet.md     && echo "✅" || echo "⬜"   # 필수
test -f docs/07-legal-checklist.md  && echo "✅" || echo "⬜"   # 필수 ★승인

# Phase 3
test -f out/video.mp4               && echo "✅" || echo "⬜"   # 필수
test -f public/audio/voiceover.mp3  && echo "✅" || echo "⬜"   # 필수
test -f public/audio/music.wav      && echo "✅" || echo "⬜"   # 필수
ls public/images/*.{png,jpg,webp} 2>/dev/null | head -1 && echo "✅" || echo "⬜"  # 선택
ls public/footage/*.{mp4,webm} 2>/dev/null | head -1 && echo "✅" || echo "⬜"     # 선택

# Phase 4
test -f docs/08-revision-log.md     && echo "✅" || echo "⬜"   # 선택
ls out/thumbnails/*.{png,jpg} 2>/dev/null | head -1 && echo "✅" || echo "⬜"      # 선택
test -f docs/10-seo-metadata.md     && echo "✅" || echo "⬜"   # 선택

# Phase 5
test -f docs/09-delivery-package.md && echo "✅" || echo "⬜"   # 필수

# Phase 6
test -f docs/99-archive.md          && echo "✅" || echo "⬜"   # 선택
```

`docs/style-frames/`와 `public/images/` 등 디렉토리형 산출물은 **내부에 파일이 1개 이상** 있어야 완료로 판정합니다 (빈 폴더는 미완료).

### Step 2: 진행률 계산

- **필수 항목**: project-scope, select-models, receive-brief, creative-brief, treatment, write-script, storyboard, prompt-sheet, legal-check, create-video, add-voiceover, add-music, review-video, qc-check, export-multi, deliver (총 16개)
- **선택 항목**: brand-kit, concept-options, fact-check, style-frame, animatic, generate-image, generate-clip, find-footage, add-captions, audio-mix, color-grade, revision-log, accessibility, thumbnail, seo-metadata, localize, repurpose, archive (총 18개)
- **현재 단계**: 마지막으로 완료된 필수 항목이 속한 Phase 기준
- **다음 명령어**: 현재 단계에서 미완료된 첫 번째 필수 항목

### Step 3: docs/00-progress.md 갱신

기존 `docs/00-progress.md`를 스캔 결과로 **덮어씁니다**. 형식:

```markdown
# 진행 상황 대시보드

> 마지막 업데이트: [현재 날짜 시간]

## 전체 요약

| 항목 | 값 |
|------|---|
| 전체 진행률 | [필수 완료 수/필수 전체 수] ([%]) |
| 필수 완료 | [n/16] |
| 선택 완료 | [n/18] |
| 현재 단계 | Phase [n] — [단계명] |
| 다음 명령어 | /[커맨드명] |

## Phase 1: 기획

| 상태 | 커맨드 | 산출물 | 구분 |
|------|--------|--------|------|
| ✅ | /project-scope | docs/00-project-scope.md | 필수 |
| ⬜ | /brand-kit | docs/00-brand-guidelines.md | 선택 |
| ✅ | /select-models | config.yaml | 필수 |

[Phase 2~6 동일 형식]

## 완료된 산출물

| 파일 | 생성일 | 크기 |
|------|--------|------|
| docs/00-project-scope.md | 2025-05-12 | 2.1KB |
| config.yaml | 2025-05-12 | 0.5KB |
| ... | ... | ... |
```

### Step 4: 터미널 출력

갱신된 내용을 터미널에도 요약 표시합니다:

```
========================================
  영상 제작 진행 상황
========================================

Phase 1: 기획               ✅ 완료 [2/2 필수]
Phase 2: 프리프로덕션        🔄 진행중 [3/8 필수]
Phase 3: 프로덕션            ⬜ 대기
Phase 4: 포스트프로덕션      ⬜ 대기
Phase 5: 납품               ⬜ 대기
Phase 6: 공개 후             ⬜ 대기

----------------------------------------
  필수: 5/16 (31%) | 선택: 1/18 (6%)
  다음: /write-script (Phase 2)
========================================
```

### Step 5: 안내

> "진행 상황을 `docs/00-progress.md`에 업데이트했습니다.
> 현재 **Phase [n] — [단계명]**이며, 다음 단계는 `/[커맨드명]`입니다. 실행할까요?"

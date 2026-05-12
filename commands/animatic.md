---
name: animatic
description: Create a rough animated preview (animatic) by combining storyboard images with timing and temporary audio. Validates pacing before full production begins.
---

# Animatic — 애니매틱 (타이밍 프리뷰)

스토리보드 이미지에 타이밍과 임시 오디오를 입혀 페이싱을 사전 검증하는 간이 프리뷰를 생성합니다.

## 사전 조건

다음 파일이 존재해야 합니다:
- `docs/04-script.md`
- `docs/05-storyboard.md`
- 스토리보드 프리뷰 이미지들 (`docs/storyboard-previews/`)

없으면 해당 커맨드를 먼저 실행하도록 안내합니다.

## 프로세스

### Step 1: 스토리보드 및 스크립트 읽기

```bash
cat docs/05-storyboard.md
cat docs/04-script.md
ls docs/storyboard-previews/ 2>/dev/null
```

### Step 2: 임시 나레이션 생성

스크립트의 나레이션 텍스트로 임시 TTS를 생성합니다.

`config.yaml`의 TTS 프로바이더를 사용합니다.

```bash
mkdir -p public/audio
```

### Step 3: Remotion 애니매틱 컴포지션 생성

각 스토리보드 이미지를 해당 장면의 타이밍에 맞춰 배치하는 간단한 Remotion 컴포지션을 생성합니다.

- 각 샷의 `시간` 정보를 프레임으로 변환
- 스토리보드 이미지를 `<Img>` 또는 `<Sequence>`로 배치
- 나레이션 오디오를 `<Audio>`로 추가
- 장면 간 단순 컷 전환 (페이드 정도)
- 화면 하단에 나레이션 자막 표시 (선택)

파일 위치: `src/Animatic.tsx`
Root.tsx에 `animatic` Composition 추가 (메인 영상과 별도)

### Step 4: 프리뷰 확인

`rules/questioning-protocol.md`의 모호함 감지 규칙을 적용합니다.

```bash
npm run dev
```

> "애니매틱 프리뷰가 준비되었습니다. http://localhost:3000 에서 'animatic' 컴포지션을 선택하여 확인해주세요.\n\n확인할 포인트:\n- 각 장면의 길이가 적절한가?\n- 나레이션과 화면 전환 타이밍이 맞는가?\n- 전체 흐름이 자연스러운가?\n- 너무 빠르거나 느린 부분이 있는가?"

### Step 5: 피드백 반영

타이밍 조정이 필요한 경우:
> "어떤 부분을 조정할까요?\n- Shot [X]: 더 길게/짧게 (현재 [n]초)\n- 전체적으로 더 빠르게/느리게\n- 특정 구간에서 멈춤(pause) 추가"

수정 후 다시 프리뷰합니다.

## 다음 단계

타이밍 확정 후 `/prompt-sheet` → `/create-video`로 진행합니다.

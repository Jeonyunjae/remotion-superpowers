---
name: style-frame
description: Generate high-quality design reference frames (3-5 key frames) that establish the final visual quality target. More refined than storyboard previews, these represent the production-quality look.
---

# Style Frame — 스타일프레임 제작

스토리보드의 프리뷰 이미지보다 높은 퀄리티의 디자인 시안 프레임을 3~5장 제작합니다. 최종 영상의 비주얼 수준을 사전에 확인하는 단계입니다.

## 사전 조건

다음 파일이 존재해야 합니다:
- `docs/03-treatment.md`
- `docs/05-storyboard.md`
- `docs/00-brand-guidelines.md` (있을 경우)

없으면 해당 커맨드를 먼저 실행하도록 안내합니다.

## 프로세스

### Step 1: 핵심 프레임 선정

```bash
cat docs/05-storyboard.md
cat docs/03-treatment.md
cat docs/00-brand-guidelines.md 2>/dev/null
```

스토리보드에서 **비주얼적으로 가장 중요한 장면 3~5개**를 선정합니다:
- 오프닝 (첫인상)
- 핵심 메시지 장면
- 클라이맥스
- 엔딩/CTA
- 가장 복잡한 비주얼 장면

### Step 2: 사용자 확인

`rules/questioning-protocol.md`의 모호함 감지 규칙을 적용합니다.

> "스토리보드에서 스타일프레임을 제작할 핵심 장면 [n]개를 선정했습니다:\n1. Shot [X]: [설명]\n2. Shot [Y]: [설명]\n3. Shot [Z]: [설명]\n다른 장면을 추가하거나 변경할까요?"

### Step 3: 고퀄리티 프롬프트 작성

각 선정 프레임에 대해 최대한 상세한 이미지 생성 프롬프트를 작성합니다:
- 트리트먼트의 색상 팔레트 반영
- 브랜드 가이드라인의 색상/스타일 반영
- 스토리보드의 구도/앵글/인물 정보 반영
- 고해상도 키워드 추가 (8k, detailed, professional)

### Step 4: AI 이미지 생성

`config.yaml`의 이미지 프로바이더를 사용합니다.

```bash
mkdir -p docs/style-frames
```

각 프레임을 `docs/style-frames/sf-[번호]-[설명].png` 형식으로 저장합니다.

### Step 5: 승인 요청

> "스타일프레임 [n]장을 생성했습니다. 이것이 최종 영상의 비주얼 기준이 됩니다. 확인해주세요.\n수정이 필요한 프레임이 있으면 번호로 말씀해주세요."

**승인 시**: `/prompt-sheet`에서 이 스타일프레임을 참조하여 프롬프트를 작성합니다.
**수정 요청 시**: 피드백 반영 후 재생성합니다.

## 다음 단계

승인 완료 후 `/prompt-sheet`를 실행합니다.

---
name: export-multi
description: Export the master video into multiple platform-specific formats — resize, reframe, adjust safe zones, and create derivative versions (16:9, 9:16, 1:1) from a single source.
---

# Export Multi — 다중 포맷 출력

하나의 마스터 영상에서 여러 플랫폼에 맞는 포맷별 버전을 생성합니다. 리프레이밍, 세이프존 조정, 길이별 컷 등을 자동화합니다.

## 사전 조건

렌더링된 마스터 영상이 존재해야 합니다:
```bash
ls out/*.mp4 2>/dev/null
```

## 프로세스

### Step 1: 필요 포맷 확인

```bash
cat docs/00-project-scope.md 2>/dev/null
```

> "어떤 플랫폼 버전이 필요한가요? (복수 선택)\n1. YouTube (16:9, 1920x1080)\n2. Instagram Reels / TikTok (9:16, 1080x1920)\n3. Instagram Feed (1:1, 1080x1080)\n4. LinkedIn (16:9, 1920x1080, 2분 이내)\n5. 웹 배너 (다양한 비율, 6~15초)\n6. GIF 프리뷰 (10초 이하)"

### Step 2: 리프레이밍 전략 결정

가로→세로 변환 시:
> "세로 영상으로 변환할 때 어떻게 리프레이밍할까요?\n1. 중앙 크롭 — 가운데만 잘라냄 (단순, 빠름)\n2. 핵심 피사체 추적 — 중요한 대상을 중심으로 크롭 (품질 높음, 수동)\n3. 상하 여백 추가 — 원본을 유지하고 위아래에 블러 배경 또는 색상 추가"

### Step 3: 포맷별 렌더링

각 포맷에 대해 Remotion 컴포지션을 생성하거나 ffmpeg으로 변환합니다.

**방법 A: Remotion 컴포지션 (품질 최적)**

각 포맷용 래퍼 컴포지션을 생성합니다:

```tsx
// src/exports/Vertical.tsx — 9:16 세로 버전
export const VerticalVersion: React.FC = () => {
  return (
    <AbsoluteFill style={{backgroundColor: '#000'}}>
      {/* 중앙 크롭 또는 리프레이밍 */}
      <div style={{
        width: 1080, height: 1920,
        overflow: 'hidden',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }}>
        <MainComposition />
      </div>
    </AbsoluteFill>
  );
};
```

Root.tsx에 각 포맷 컴포지션 등록:
```tsx
<Composition id="vertical" component={VerticalVersion} width={1080} height={1920} fps={30} durationInFrames={...} />
<Composition id="square" component={SquareVersion} width={1080} height={1080} fps={30} durationInFrames={...} />
```

```bash
npx remotion render vertical out/video-vertical.mp4
npx remotion render square out/video-square.mp4
```

**방법 B: ffmpeg 변환 (빠름)**

```bash
# 세로 (중앙 크롭)
ffmpeg -i out/video.mp4 -vf "crop=ih*9/16:ih,scale=1080:1920" -c:a copy out/video-vertical.mp4

# 정방형 (중앙 크롭)
ffmpeg -i out/video.mp4 -vf "crop=ih:ih,scale=1080:1080" -c:a copy out/video-square.mp4

# 짧은 버전 (처음 60초)
ffmpeg -i out/video.mp4 -t 60 -c copy out/video-60s.mp4

# GIF (처음 10초, 480px)
ffmpeg -i out/video.mp4 -t 10 -vf "fps=10,scale=480:-1:flags=lanczos" -loop 0 out/preview.gif
```

### Step 4: 세이프존 확인

각 포맷에서 텍스트/자막이 잘리지 않는지 확인합니다:
- YouTube: 상하좌우 5% 여백
- Instagram: 상단 10%, 하단 15% (UI 오버레이 영역)
- TikTok: 상단 15%, 하단 20%, 우측 15% (버튼 영역)

### Step 5: 출력 목록 제시

```
📦 Export Complete

| # | 포맷 | 파일 | 해상도 | 길이 | 크기 |
|---|------|------|--------|------|------|
| 1 | YouTube (가로) | out/video.mp4 | 1920x1080 | [시간] | [크기] |
| 2 | Reels/TikTok (세로) | out/video-vertical.mp4 | 1080x1920 | [시간] | [크기] |
| 3 | Instagram Feed (정방형) | out/video-square.mp4 | 1080x1080 | [시간] | [크기] |
| 4 | GIF 프리뷰 | out/preview.gif | 480xAuto | 10초 | [크기] |

총 [n]개 파일 생성 완료.
```

## 다음 단계

포맷 출력 완료 후 `/deliver`로 납품 패키지를 생성합니다.

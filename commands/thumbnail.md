---
name: thumbnail
description: Generate click-optimized video thumbnails — extract key frames, create AI-enhanced designs with text overlays, and produce multiple variants for A/B testing.
---

# Thumbnail — 썸네일 디자인

영상의 핵심 장면에서 썸네일을 추출하고, 텍스트 오버레이와 AI 보정을 적용하여 클릭률(CTR)을 높이는 썸네일을 생성합니다.

## 프로세스

### Step 1: 영상 정보 확인

```bash
cat docs/01-client-brief.md 2>/dev/null
cat docs/04-script.md 2>/dev/null
ls out/*.mp4 2>/dev/null
```

### Step 2: 썸네일 방향 질문

`rules/questioning-protocol.md`의 모호함 감지 규칙을 적용합니다.

| # | 질문 |
|---|------|
| 1 | "썸네일에 텍스트를 넣을 건가요?\n1. 네 — 핵심 키워드/제목 오버레이\n2. 아니오 — 이미지만\n3. 로고만 추가" |
| 2 | "썸네일 스타일은?\n1. 영상 캡처 기반 (실제 장면에서 추출)\n2. AI 생성 (별도 디자인)\n3. 혼합 (캡처 + 그래픽 요소)" |
| 3 | "A/B 테스트용 복수 시안이 필요한가요?\n1. 1장이면 충분\n2. 2~3장 시안 비교" |

### Step 3: 핵심 프레임 추출

렌더링된 영상에서 시각적으로 임팩트 있는 프레임을 추출합니다:

```bash
mkdir -p out/thumbnails

# 영상에서 주요 장면 프레임 추출 (10초 간격)
ffmpeg -i out/video.mp4 -vf "fps=1/10,scale=1280:720" -q:v 2 out/thumbnails/frame-%03d.jpg 2>/dev/null

# 또는 특정 타임코드에서 추출
ffmpeg -i out/video.mp4 -ss 00:00:03 -frames:v 1 -q:v 2 out/thumbnails/candidate-1.jpg
ffmpeg -i out/video.mp4 -ss 00:00:15 -frames:v 1 -q:v 2 out/thumbnails/candidate-2.jpg
```

### Step 4: 썸네일 제작

**옵션 A: 프레임 기반 (텍스트 오버레이)**

Remotion으로 썸네일 컴포지션을 생성합니다:

```tsx
// src/Thumbnail.tsx
export const Thumbnail: React.FC<{text: string}> = ({text}) => {
  return (
    <AbsoluteFill style={{backgroundColor: '#000'}}>
      <Img src={staticFile("thumbnails/candidate-1.jpg")} style={{width: '100%', height: '100%', objectFit: 'cover'}} />
      {/* 텍스트 오버레이 */}
      <div style={{
        position: 'absolute', bottom: 40, left: 40, right: 40,
        fontSize: 64, fontWeight: 900, color: '#FFF',
        textShadow: '3px 3px 6px rgba(0,0,0,0.8)',
        lineHeight: 1.2,
      }}>
        {text}
      </div>
    </AbsoluteFill>
  );
};
```

```bash
# 정적 이미지로 렌더링
npx remotion still Thumbnail out/thumbnails/final-thumbnail.png --props='{"text":"[제목]"}'
```

**옵션 B: AI 생성**

`config.yaml`의 이미지 프로바이더를 사용하여 별도의 썸네일 이미지를 생성합니다.

### Step 5: 사양 확인

**YouTube 썸네일 사양:**
- 해상도: 1280 x 720 (최소 640 x 360)
- 비율: 16:9
- 파일 크기: 2MB 이하
- 포맷: JPG, PNG, GIF, BMP

```bash
# 사양 확인
identify out/thumbnails/final-thumbnail.png 2>/dev/null || ffprobe -v quiet -show_entries stream=width,height out/thumbnails/final-thumbnail.png
ls -lh out/thumbnails/final-thumbnail.png
```

### Step 6: 결과 제시

> "썸네일 [n]장을 생성했습니다:\n\n1. `out/thumbnails/final-thumbnail.png` — [설명]\n2. `out/thumbnails/final-thumbnail-2.png` — [설명]\n\n사양: 1280x720, [크기]\n\n마음에 드는 것을 선택해주세요. 텍스트나 구도를 수정할 수도 있습니다."

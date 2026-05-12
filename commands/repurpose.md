---
name: repurpose
description: Repurpose a completed video into derivative content — extract highlight clips, create GIFs, generate blog posts from transcripts, pull social media assets, and extract audio for podcasts.
---

# Repurpose — 콘텐츠 재활용

완성된 영상에서 다양한 파생 콘텐츠를 생성합니다. 하이라이트 클립, GIF, 블로그 포스트, SNS 에셋, 오디오 추출 등을 포함합니다.

## 사전 조건

렌더링된 영상과 스크립트가 존재해야 합니다:
```bash
ls out/*.mp4
cat docs/04-script.md 2>/dev/null
```

## 프로세스

### Step 1: 재활용 유형 선택

> "어떤 파생 콘텐츠가 필요한가요? (복수 선택)\n1. 하이라이트 클립 (핵심 장면 15~60초 추출)\n2. 숏폼 클립 (세로 9:16 Reels/TikTok)\n3. GIF 프리뷰 (SNS 공유용 움짤)\n4. 블로그 포스트 (나레이션 기반 텍스트 콘텐츠)\n5. 오디오만 추출 (팟캐스트용)\n6. 정적 이미지 카드 (SNS 카드뉴스용)\n7. 전체 다 생성"

### Step 2: 핵심 구간 식별

스크립트와 영상을 분석하여 임팩트 있는 구간을 식별합니다:

```bash
cat docs/04-script.md
cat docs/05-storyboard.md 2>/dev/null
```

> "하이라이트 클립으로 추출할 구간을 추천합니다:\n1. [시간 구간] — [내용 요약]\n2. [시간 구간] — [내용 요약]\n3. [시간 구간] — [내용 요약]\n\n다른 구간을 원하시면 알려주세요."

### Step 3: 파생 콘텐츠 생성

```bash
mkdir -p out/repurposed
```

**하이라이트 클립:**
```bash
ffmpeg -i out/video.mp4 -ss [start] -t [duration] -c copy out/repurposed/highlight-1.mp4
```

**숏폼 클립 (세로):**
```bash
ffmpeg -i out/video.mp4 -ss [start] -t [duration] -vf "crop=ih*9/16:ih,scale=1080:1920" out/repurposed/short-1.mp4
```

**GIF:**
```bash
ffmpeg -i out/video.mp4 -ss [start] -t 5 -vf "fps=10,scale=480:-1:flags=lanczos" -loop 0 out/repurposed/preview.gif
```

**오디오 추출:**
```bash
ffmpeg -i out/video.mp4 -vn -acodec mp3 -ab 192k out/repurposed/audio.mp3
```

**블로그 포스트:**
스크립트의 나레이션 텍스트를 기반으로 블로그 형식의 텍스트를 생성합니다.

**이미지 카드:**
영상의 핵심 프레임을 추출하고 텍스트 오버레이를 추가합니다.
```bash
ffmpeg -i out/video.mp4 -ss [time] -frames:v 1 -q:v 2 out/repurposed/card-1.jpg
```

### Step 4: 결과 정리

```
♻️ Repurpose Complete

| # | 유형 | 파일 | 설명 |
|---|------|------|------|
| 1 | 하이라이트 클립 | out/repurposed/highlight-1.mp4 | [구간], [길이] |
| 2 | 숏폼 클립 | out/repurposed/short-1.mp4 | 세로 9:16 |
| 3 | GIF | out/repurposed/preview.gif | [크기] |
| 4 | 오디오 | out/repurposed/audio.mp3 | MP3, 192kbps |
| 5 | 블로그 포스트 | out/repurposed/blog-post.md | [글자수] |
| 6 | 이미지 카드 | out/repurposed/card-*.jpg | [장수] |
```

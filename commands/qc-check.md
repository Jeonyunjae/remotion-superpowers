---
name: qc-check
description: Automated technical quality control using ffprobe — verify codec, resolution, frame rate, audio channels, loudness (LUFS), file size limits, and platform compliance.
---

# QC Check — 기술 품질 검증

ffprobe/ffmpeg 기반으로 렌더링된 영상의 기술 사양을 자동 검증합니다. 코덱, 해상도, 프레임레이트, 오디오 채널, LUFS, 파일 크기 등을 플랫폼 기준에 맞춰 확인합니다.

## 사전 조건

렌더링된 영상 파일이 존재해야 합니다:
```bash
ls out/*.mp4 out/*.webm out/*.mov 2>/dev/null
```

## 플랫폼별 기술 사양

| 항목 | YouTube | Instagram Reels | TikTok | LinkedIn | 사내 |
|------|---------|----------------|--------|----------|------|
| 최대 파일 크기 | 256GB | 650MB | 287.6MB | 5GB | 제한 없음 |
| 최대 길이 | 12시간 | 90초 | 10분 | 10분 | 제한 없음 |
| 권장 코덱 | H.264 | H.264 | H.264 | H.264 | H.264 |
| 권장 해상도 (가로) | 1920x1080 | 1080x1920 | 1080x1920 | 1920x1080 | 1920x1080 |
| 권장 FPS | 30 | 30 | 30 | 30 | 30 |
| 오디오 | AAC, 48kHz | AAC | AAC | AAC | AAC |
| 오디오 채널 | 스테레오 | 스테레오 | 스테레오 | 스테레오 | 스테레오 |
| 목표 LUFS | -14 | -14 | -14 | -14 | -16~-20 |

## 프로세스

### Step 1: 타깃 플랫폼 확인

```bash
cat docs/00-project-scope.md 2>/dev/null
cat docs/01-client-brief.md 2>/dev/null
```

배포 채널 정보에서 타깃 플랫폼을 확인합니다. 없으면 질문합니다:

> "QC 검증 대상 플랫폼은 어디인가요?\n1. YouTube\n2. Instagram Reels/TikTok\n3. LinkedIn\n4. 사내 인트라넷\n5. 복수 플랫폼 (모든 기준 충족)"

### Step 2: 기술 정보 추출

```bash
# 비디오 정보
ffprobe -v quiet -print_format json -show_format -show_streams out/video.mp4

# 파일 크기
ls -lh out/video.mp4

# LUFS 측정
ffmpeg -i out/video.mp4 -af loudnorm=print_format=json -f null - 2>&1 | grep -A 20 "Parsed_loudnorm"
```

### Step 3: 검증 항목별 판정

각 항목을 타깃 플랫폼 기준에 대조하여 PASS/WARN/FAIL로 판정합니다.

### Step 4: QC 리포트 출력

```
📋 QC Report: [filename]

Platform Target: [platform]

| 항목 | 기준 | 실제 | 판정 |
|------|------|------|------|
| 코덱 | H.264 | [actual] | [PASS/FAIL] |
| 해상도 | [expected] | [actual] | [PASS/FAIL] |
| 프레임레이트 | [expected] fps | [actual] fps | [PASS/FAIL] |
| 비트레이트 | 적정 범위 | [actual] kbps | [PASS/WARN/FAIL] |
| 오디오 코덱 | AAC | [actual] | [PASS/FAIL] |
| 오디오 샘플레이트 | 48000 Hz | [actual] Hz | [PASS/WARN] |
| 오디오 채널 | 스테레오 | [actual] | [PASS/WARN] |
| Integrated LUFS | [target] | [actual] | [PASS/WARN/FAIL] |
| True Peak | < [limit] dBTP | [actual] dBTP | [PASS/FAIL] |
| 파일 크기 | < [limit] | [actual] | [PASS/WARN/FAIL] |
| 영상 길이 | < [limit] | [actual] | [PASS/FAIL] |

Overall: [PASS / PASS with warnings / FAIL]

[FAIL 항목에 대한 수정 가이드]
```

### Step 5: 자동 수정 제안

FAIL 항목이 있으면 수정 방법을 제안합니다:

- 코덱 불일치: `npx remotion render [id] out/video.mp4 --codec h264`
- 해상도 불일치: Root.tsx에서 width/height 수정
- LUFS 불일치: `/audio-mix` 재실행 권고
- 파일 크기 초과: CRF 값 조정 `--crf [higher value]`
- 프레임레이트 불일치: Root.tsx에서 fps 수정

## 다음 단계

QC PASS 후 `/deliver` 또는 `/export-multi`로 진행합니다.

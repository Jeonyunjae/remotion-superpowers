---
name: audio-mix
description: Apply professional audio mixing standards — LUFS normalization, dialogue-to-music balance, ducking optimization, and platform-specific loudness targeting.
---

# Audio Mix — 오디오 믹싱 및 마스터링

플랫폼별 라우드니스 표준(LUFS)을 적용하고, 나레이션/배경음악/효과음 간 밸런스를 최적화합니다.

## 사전 조건

Remotion 프로젝트에 오디오 파일이 존재해야 합니다:
```bash
ls public/audio/
```

## 배경 지식: LUFS 표준

| 플랫폼 | 목표 LUFS | True Peak 상한 | 비고 |
|--------|----------|---------------|------|
| YouTube | -14 LUFS | -1 dBTP | 자동 라우드니스 정규화 적용 |
| Instagram/TikTok | -14 LUFS | -1 dBTP | 모바일 최적화 |
| 방송 (EBU R128) | -23 LUFS | -1 dBTP | 유럽 표준 |
| 방송 (ATSC A/85) | -24 LKFS | -2 dBTP | 미국 표준 |
| 팟캐스트 | -16 LUFS | -1 dBTP | Apple Podcasts 권장 |
| 사내 프레젠테이션 | -16 ~ -20 LUFS | -1 dBTP | 조용한 환경 |

## 프로세스

### Step 1: 타깃 플랫폼 확인

```bash
cat docs/00-project-scope.md 2>/dev/null
cat docs/01-client-brief.md 2>/dev/null
```

> "영상의 주요 배포 플랫폼은 어디인가요?\n1. YouTube (-14 LUFS)\n2. Instagram/TikTok (-14 LUFS)\n3. 사내 인트라넷/프레젠테이션 (-16~-20 LUFS)\n4. 방송 (-23 LUFS)\n5. 복수 플랫폼 (가장 엄격한 기준 적용)"

### Step 2: 현재 오디오 레벨 측정

ffmpeg의 loudnorm 필터로 현재 오디오 레벨을 측정합니다:

```bash
# 나레이션 레벨 측정
ffmpeg -i public/audio/voiceover.mp3 -af loudnorm=print_format=json -f null - 2>&1 | grep -A 20 "Parsed_loudnorm"

# 배경음악 레벨 측정
ffmpeg -i public/audio/music.wav -af loudnorm=print_format=json -f null - 2>&1 | grep -A 20 "Parsed_loudnorm"
```

### Step 3: 오디오 밸런스 규칙 적용

**대사 vs BGM 밸런스:**
- 나레이션 구간: BGM은 나레이션보다 **10~15dB 낮게**
- 나레이션 없는 구간: BGM 원래 레벨로 복귀
- 전환 구간: 0.5~1초 페이드로 부드럽게

**더킹(Ducking) 설정:**
- 나레이션 시작 0.3초 전부터 BGM 감쇄 시작
- 나레이션 종료 0.5초 후 BGM 복귀
- 감쇄량: -10 ~ -15dB (나레이션 명료도에 따라 조정)

**효과음 밸런스:**
- 효과음은 나레이션보다 낮되, BGM보다는 높게
- 효과음 없는 구간에서 갑자기 큰 효과음이 나오지 않도록

### Step 4: Remotion 오디오 코드 최적화

Remotion 컴포지션의 오디오 볼륨 설정을 업데이트합니다:

```tsx
// 나레이션: 기준 볼륨
<Audio src={staticFile("audio/voiceover.mp3")} volume={1.0} />

// BGM: 나레이션 구간에서 더킹
<Audio
  src={staticFile("audio/music.wav")}
  volume={(f) => {
    // 나레이션 구간에서 0.15 (-16dB), 아닌 구간에서 0.5 (-6dB)
    return isNarrationPlaying(f) ? 0.15 : 0.5;
  }}
/>

// 효과음: 타이밍에 맞춰 배치
<Audio src={staticFile("audio/sfx/effect.mp3")} volume={0.7} startFrom={0} />
```

### Step 5: 렌더링 후 LUFS 검증

영상 렌더링 후 최종 LUFS를 측정합니다:

```bash
ffmpeg -i out/video.mp4 -af loudnorm=print_format=json -f null - 2>&1 | grep -A 20 "Parsed_loudnorm"
```

**검증 기준:**
- Integrated Loudness가 목표 LUFS ± 1dB 이내
- True Peak이 상한 이하
- Loudness Range(LRA)가 적정 범위 (교육: 5~10 LU, 광고: 3~8 LU)

### Step 6: 리포트 출력

> "🔊 오디오 믹싱 결과\n\n목표: [platform] (-[X] LUFS)\n측정 결과:\n- Integrated Loudness: [X] LUFS [OK/경고]\n- True Peak: [X] dBTP [OK/경고]\n- Loudness Range: [X] LU [OK/경고]\n- 나레이션 명료도: [양호/조정 필요]\n\n[조정이 필요한 경우 구체적 제안]"

## 다음 단계

오디오 믹싱 완료 후 `/review-video` 또는 `/qc-check`로 진행합니다.

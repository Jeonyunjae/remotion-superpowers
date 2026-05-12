---
name: color-grade
description: Check color consistency across AI-generated images and video clips. Suggest color correction and LUT application to unify the visual look throughout the video.
---

# Color Grade — 컬러 일관성 검사 및 보정

AI로 생성된 이미지와 영상 클립 간의 색감 불일치를 탐지하고, 통일된 비주얼 룩을 위한 컬러 보정을 제안합니다.

## 사전 조건

다음이 존재해야 합니다:
- `docs/03-treatment.md` (색상 팔레트, 조명 방향 참조)
- `docs/00-brand-guidelines.md` (브랜드 색상 참조, 있을 경우)
- 생성된 이미지/영상 에셋 (`public/images/`, `public/footage/`)

## 프로세스

### Step 1: 참조 문서 읽기

```bash
cat docs/03-treatment.md
cat docs/00-brand-guidelines.md 2>/dev/null
ls public/images/ public/footage/ 2>/dev/null
```

### Step 2: 색감 분석

각 이미지/영상 클립의 색감 특성을 분석합니다:

```bash
# 각 이미지의 색상 히스토그램/평균 색상 추출
for img in public/images/*.{jpg,png}; do
  echo "=== $img ==="
  ffprobe -v quiet -show_entries frame_tags=lavfi.signalstats.YAVG,lavfi.signalstats.UAVG,lavfi.signalstats.VAVG -f lavfi -i "movie=$img,signalstats" 2>/dev/null | head -10
done
```

### Step 3: 일관성 점검

**점검 항목:**
- 전체적인 색온도(화이트밸런스)가 장면 간 일치하는가?
- 밝기/대비가 급격히 변하는 장면이 있는가?
- 트리트먼트에서 정한 색상 팔레트와 실제 에셋이 일치하는가?
- 브랜드 색상이 정확하게 재현되는가?
- AI 생성 이미지 특유의 과포화/부자연스러운 색감이 있는가?

### Step 4: 보정 제안

불일치가 발견되면 Remotion에서 CSS 필터로 보정하는 방법을 제안합니다:

```tsx
// 색온도 조정 (웜하게)
<div style={{ filter: 'sepia(0.1) saturate(1.1)' }}>
  <Img src={staticFile("images/scene3.jpg")} />
</div>

// 밝기/대비 조정
<div style={{ filter: 'brightness(1.05) contrast(1.1)' }}>
  <Img src={staticFile("images/scene5.jpg")} />
</div>

// 전체 영상에 통일된 LUT 효과 (CSS)
<AbsoluteFill style={{
  filter: 'contrast(1.05) saturate(0.95) brightness(1.02)',
  mixBlendMode: 'normal',
}}>
  {children}
</AbsoluteFill>
```

### Step 5: 결과 리포트

> "🎨 컬러 일관성 검사 결과\n\n분석한 에셋: [n]개\n\n| 에셋 | 색온도 | 밝기 | 채도 | 판정 |\n|------|--------|------|------|------|\n| [파일명] | [웜/뉴트럴/쿨] | [어두움/적정/밝음] | [저/적정/고] | [OK/조정필요] |\n\n보정 제안:\n- [구체적 보정 내용]\n\n적용할까요?"

## 다음 단계

컬러 보정 완료 후 `/review-video` 또는 `/qc-check`로 진행합니다.

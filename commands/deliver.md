---
name: deliver
description: Create the final delivery package — organize all output files, generate a spec sheet, compile asset credits, and produce a delivery document with usage guidelines.
---

# Deliver — 최종 납품 패키지 생성

모든 최종 파일을 정리하고, 스펙시트, 에셋 크레딧, 사용 가이드를 포함한 납품 패키지를 생성합니다.

## 사전 조건

다음이 완료되어야 합니다:
- 최종 영상 렌더링 완료 (`out/` 폴더)
- QC 검증 완료 (권장)
- 법적 체크리스트 확인 완료 (권장)

## 프로세스

### Step 1: 최종 파일 확인

```bash
ls -la out/
cat docs/07-legal-checklist.md 2>/dev/null
cat docs/08-revision-log.md 2>/dev/null
cat docs/10-seo-metadata.md 2>/dev/null
```

### Step 2: 납품 폴더 구성

```bash
mkdir -p delivery/videos
mkdir -p delivery/thumbnails
mkdir -p delivery/subtitles
mkdir -p delivery/documents

# 최종 영상 복사
cp out/video*.mp4 delivery/videos/
cp out/thumbnails/*.png delivery/thumbnails/ 2>/dev/null

# 자막 파일 복사
cp docs/*.srt delivery/subtitles/ 2>/dev/null

# 문서 복사
cp docs/10-seo-metadata.md delivery/documents/ 2>/dev/null
```

### Step 3: 스펙시트 생성

각 영상 파일의 기술 정보를 자동 추출합니다:

```bash
for f in delivery/videos/*.mp4; do
  echo "=== $(basename $f) ==="
  ffprobe -v quiet -print_format json -show_format -show_streams "$f" | grep -E '"codec_name"|"width"|"height"|"r_frame_rate"|"duration"|"bit_rate"|"channels"|"sample_rate"'
  ls -lh "$f" | awk '{print "File size:", $5}'
  echo "---"
done
```

### Step 4: 납품 문서 작성

**출력 파일**: `docs/09-delivery-package.md`

```markdown
# Delivery Package

## 납품일
[날짜]

## 납품 파일 목록

### 영상
| # | 파일명 | 포맷 | 해상도 | FPS | 코덱 | 길이 | 크기 |
|---|--------|------|--------|-----|------|------|------|
| 1 | [파일명] | MP4 | [해상도] | [fps] | H.264 | [시간] | [크기] |
| 2 | ... | ... | ... | ... | ... | ... | ... |

### 썸네일
| # | 파일명 | 해상도 | 크기 |
|---|--------|--------|------|
| 1 | [파일명] | 1280x720 | [크기] |

### 자막
| # | 파일명 | 언어 | 포맷 |
|---|--------|------|------|
| 1 | [파일명] | [언어] | SRT |

### SEO 메타데이터
- `10-seo-metadata.md` — 제목, 설명, 태그, 챕터 마커

## 에셋 크레딧 / 라이선스 정보

| # | 에셋 유형 | 프로바이더 | 라이선스 | 크레딧 표시 |
|---|----------|-----------|---------|------------|
| 1 | 나레이션 (TTS) | [provider] | [license] | [필요/불필요] |
| 2 | 배경음악 | [provider] | [license] | [필요/불필요] |
| 3 | AI 이미지 | [provider] | [license] | [필요/불필요] |
| 4 | AI 영상 | [provider] | [license] | [필요/불필요] |
| 5 | 스톡 미디어 | [provider] | [license] | [필요/불필요] |
| 6 | 효과음 | [provider] | [license] | [필요/불필요] |

## 크레딧 표시 텍스트
[영상 설명 또는 크레딧 화면에 포함해야 할 저작자 표시]

## 사용 가이드라인
- 배포 채널: [승인된 채널 목록]
- 사용 기간: [기간]
- 수정 권한: [수정 가능 여부]
- 2차 사용: [파생물 제작 가능 여부]

## QC 결과 요약
- 최종 검증일: [날짜]
- 검증 결과: [PASS/PASS with warnings]
- 미해결 사항: [있으면 기재]

## 프로젝트 통계
- 총 작업 기간: [기간]
- 수정 라운드: [n]회
- 사용 AI 프리셋: [preset]
- 총 AI 사용 비용: [금액]
```

### Step 5: 사용자 확인

> "납품 패키지가 준비되었습니다.\n\n`delivery/` 폴더 구조:\n```\ndelivery/\n├── videos/        [영상 파일들]\n├── thumbnails/    [썸네일]\n├── subtitles/     [자막 파일]\n└── documents/     [메타데이터, 스펙시트]\n```\n\n납품 문서: `docs/09-delivery-package.md`\n\n확인해주세요."

## 다음 단계

납품 후 `/archive`로 프로젝트를 아카이빙하거나, `/repurpose`로 콘텐츠를 재활용합니다.

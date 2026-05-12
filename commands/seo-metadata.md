---
name: seo-metadata
description: Generate optimized SEO metadata for video distribution — titles, descriptions, tags, hashtags, chapter markers, and platform-specific formatting.
---

# SEO Metadata — SEO 메타데이터 생성

영상 배포를 위한 최적화된 제목, 설명, 태그, 해시태그, 챕터 마커를 생성합니다.

## 사전 조건

다음 파일을 참조합니다:
- `docs/01-client-brief.md`
- `docs/02-creative-brief.md`
- `docs/04-script.md`

## 프로세스

### Step 1: 콘텐츠 분석

```bash
cat docs/01-client-brief.md
cat docs/02-creative-brief.md
cat docs/04-script.md
```

### Step 2: 플랫폼 확인

> "메타데이터를 생성할 플랫폼을 선택해주세요 (복수 선택 가능):\n1. YouTube\n2. Instagram Reels\n3. TikTok\n4. LinkedIn\n5. 자사 웹사이트\n6. 전체"

### Step 3: 메타데이터 생성

**출력 파일**: `docs/10-seo-metadata.md`

```markdown
# SEO Metadata

## YouTube

### 제목 (최대 100자, 핵심 키워드 앞배치)
**옵션 A**: [제목안 1]
**옵션 B**: [제목안 2]
**옵션 C**: [제목안 3]

### 설명 (최대 5000자)
```
[처음 2줄에 핵심 내용 — 접혀 보이는 영역]

[본문 — 영상 내용 요약, 키워드 자연 포함]

📌 챕터:
0:00 [챕터 1 제목]
0:30 [챕터 2 제목]
1:00 [챕터 3 제목]

🔗 관련 링크:
- [링크 1]
- [링크 2]

#해시태그1 #해시태그2 #해시태그3
```

### 태그 (500자 이내, 쉼표 구분)
[키워드1], [키워드2], [키워드3], ...

### 챕터 마커 (타임스탬프)
| 시간 | 챕터 제목 |
|------|----------|
| 0:00 | [제목] |
| 0:30 | [제목] |
| ... | ... |

### 카테고리
[YouTube 카테고리 선택]

---

## Instagram Reels

### 캡션 (최대 2200자)
[훅 + 본문 + CTA + 해시태그]

### 해시태그 (최대 30개)
[관련성 높은 순서로 나열]

---

## TikTok

### 캡션 (최대 4000자)
[훅 문장 + 해시태그]

### 해시태그 (3~5개 권장)
[트렌드 + 니치 혼합]

---

## LinkedIn

### 게시글 텍스트
[전문적 톤 + CTA]

### 해시태그 (3~5개)
[업계 키워드]
```

### Step 4: 사용자 확인

> "SEO 메타데이터를 생성했습니다. 제목 옵션 중 선호하는 것을 골라주세요. 수정할 부분이 있으면 말씀해주세요."

## 다음 단계

메타데이터 확정 후 `/deliver` 또는 `/upload`(있을 경우)으로 진행합니다.

---
name: legal-check
description: Verify legal compliance for all assets used in the video — AI-generated content licenses, stock media rights, music licensing, brand/trademark usage, and attribution requirements.
---

# Legal Check — 법적 체크리스트

영상에 사용되는 모든 에셋의 법적 적합성을 검증합니다. AI 생성물의 상업적 사용 가능 여부, 스톡 미디어 라이선스, 음악 저작권, 상표 사용 등을 확인합니다.

## 사전 조건

다음 파일이 존재해야 합니다:
- `docs/00-project-scope.md` (소유권/사용 범위 확인)
- `docs/06-model-selection.md` (사용 프로바이더 확인)
- `config.yaml`

없으면 해당 커맨드를 먼저 실행하도록 안내합니다.

## 프로세스

### Step 1: 사용 에셋 목록 작성

```bash
cat docs/06-model-selection.md 2>/dev/null
cat docs/06-prompt-sheet.md 2>/dev/null
cat config.yaml
ls public/audio/ public/images/ public/footage/ public/brand/ 2>/dev/null
```

### Step 2: 프로바이더별 라이선스 확인

각 AI 프로바이더의 상업적 사용 조건을 확인합니다:

**AI 생성물 라이선스 (프로바이더별):**

| 프로바이더 | 상업적 사용 | 소유권 | 크레딧 필요 | 주의사항 |
|-----------|------------|--------|------------|----------|
| edge-tts | 가능 (Microsoft ToS) | 사용자 | 불필요 | 음성 합성물에 대한 제한 확인 |
| MusicGen (Meta) | 가능 (CC BY-NC 4.0) | 사용자 | 필요 | **비상업 라이선스 — 상업 사용 시 주의** |
| Pixazo | ToS 확인 필요 | 플랫폼별 | 확인 필요 | 서비스 약관 변경 가능 |
| Google AI Studio/Veo | ToS 확인 필요 | 확인 필요 | 확인 필요 | 무료 티어 제한 확인 |
| Pexels | 가능 (Pexels License) | 원작자 | 불필요 (권장) | 인물 사진 초상권 별도 |
| Freesound | CC 라이선스별 상이 | 원작자 | **필요 (대부분)** | CC BY, CC BY-NC 등 혼재 |
| Replicate | 모델별 상이 | 확인 필요 | 모델별 | 각 모델의 라이선스 개별 확인 |
| ElevenLabs | 유료 플랜에서 가능 | 사용자 | 불필요 | 무료 티어 제한 있음 |
| Suno (KIE) | ToS 확인 필요 | 확인 필요 | 확인 필요 | - |

### Step 3: 사용자에게 확인 질문

`rules/questioning-protocol.md`의 모호함 감지 규칙을 적용합니다.

| # | 질문 |
|---|------|
| 1 | "이 영상을 상업적으로 사용할 예정인가요? (광고, 판매, 유료 콘텐츠 등)" |
| 2 | "영상에 실존 인물의 모습이나 목소리가 포함되나요?" |
| 3 | "특정 브랜드의 로고나 제품이 영상에 등장하나요?" |
| 4 | "영상이 공개될 국가/지역은 어디인가요?" |

### Step 4: 법적 체크리스트 문서 작성

```bash
mkdir -p docs
```

**출력 파일**: `docs/07-legal-checklist.md`

```markdown
# Legal Checklist

## 프로젝트 사용 범위
- 상업적 사용: [예/아니오]
- 공개 범위: [내부/외부/글로벌]
- 사용 기간: [기간]

## AI 생성물 라이선스 검증

| # | 에셋 유형 | 프로바이더 | 라이선스 | 상업적 사용 | 크레딧 필요 | 상태 |
|---|----------|-----------|---------|------------|------------|------|
| 1 | TTS 나레이션 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |
| 2 | 배경음악 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |
| 3 | AI 이미지 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |
| 4 | AI 영상 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |
| 5 | 스톡 미디어 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |
| 6 | 효과음 | [provider] | [license] | [가능/불가] | [필요/불필요] | [OK/경고/차단] |

## 크레딧/저작자 표시 필요 목록
- [에셋 1]: [필요한 크레딧 형식]
- [에셋 2]: [필요한 크레딧 형식]

## 리스크 항목
| # | 리스크 | 심각도 | 대응 방안 |
|---|--------|--------|----------|
| 1 | [리스크 설명] | [높음/중간/낮음] | [대응 방법] |

## 최종 판정
- [ ] 모든 에셋의 라이선스가 사용 범위에 적합
- [ ] 필요한 크레딧이 영상 또는 설명에 포함 예정
- [ ] 리스크 항목에 대한 대응 방안 수립 완료
- **판정**: [진행 가능 / 일부 수정 후 진행 / 중단 필요]
```

### Step 5: 경고 및 권고

차단(Block) 항목이 있으면:
> "법적 리스크가 발견되었습니다:\n[리스크 내용]\n\n대안:\n1. [대안 프로바이더/에셋 사용]\n2. [라이선스 업그레이드]\n3. [해당 에셋 제거]"

## 다음 단계

법적 검증 완료 후 `/create-video`로 제작을 시작합니다.

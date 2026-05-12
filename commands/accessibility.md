---
name: accessibility
description: Verify video accessibility compliance — closed captions (not just styled subtitles), audio descriptions, color contrast ratios, flash/flicker detection, and WCAG 2.1 AA standards.
---

# Accessibility — 접근성 검증

영상의 접근성 표준(WCAG 2.1 AA) 준수 여부를 검증합니다. 자막, 색상 대비, 깜빡임, 오디오 디스크립션 등을 확인합니다.

## 배경

- **WCAG 2.1 Level AA**: 국제 웹 접근성 표준
- 미국 ADA, 유럽 EAA, 한국 장애인차별금지법 등에서 영상 접근성 요구
- 자동 생성 자막만으로는 접근성 기준 미충족 (비언어 오디오 설명 필요)

## 프로세스

### Step 1: 접근성 요구 수준 확인

> "이 영상의 접근성 요구 수준은?\n1. 기본 — 자막만 (개인 프로젝트, SNS)\n2. 표준 — 자막 + 색상 대비 (기업 외부 공개)\n3. 엄격 — 풀 WCAG 2.1 AA 준수 (공공기관, 법적 의무)\n4. 최고 — WCAG 2.1 AAA (장애인 전용 서비스)"

### Step 2: 자막 접근성 검증

**현재 자막 확인:**
```bash
# SRT 파일 확인
ls docs/*.srt public/audio/*.srt 2>/dev/null
# Remotion 캡션 컴포넌트 확인
grep -r "Caption\|caption\|subtitle" src/ 2>/dev/null
```

**클로즈드 캡션 체크리스트:**
- [ ] 모든 대사/나레이션이 자막에 포함
- [ ] 비언어 오디오 설명 포함 (예: [배경 음악], [박수], [사이렌 소리])
- [ ] 화자 식별 표시 (복수 화자인 경우)
- [ ] 동기화 정확도 (오디오와 자막 타이밍 ±0.5초 이내)
- [ ] 한 번에 2줄 이하, 한 줄 42자(영문) / 21자(한글) 이내

### Step 3: 색상 대비 검증

영상 내 텍스트 오버레이의 색상 대비를 확인합니다.

**WCAG 기준:**
- 일반 텍스트: 최소 대비 **4.5:1** (AA)
- 큰 텍스트 (18pt+): 최소 대비 **3:1** (AA)
- AAA 등급: 7:1 / 4.5:1

Remotion 코드에서 텍스트 색상과 배경 색상을 추출하여 대비율을 계산합니다.

```bash
# 텍스트 스타일 추출
grep -r "color:\|backgroundColor:\|background:" src/ | grep -v node_modules
```

### Step 4: 깜빡임/플래시 검증

**WCAG 기준**: 초당 3회 이상 깜빡임 금지 (광과민성 발작 위험)

Remotion 코드에서 빠른 opacity 변화나 밝기 변화를 탐지합니다:
```bash
# 빠른 깜빡임 패턴 탐지
grep -r "opacity\|flash\|blink\|strobe" src/ | grep -v node_modules
```

### Step 5: 접근성 리포트

```markdown
# Accessibility Report

## 검증 수준: [기본/표준/엄격/최고]

## 자막 (Captions)
| 항목 | 기준 | 판정 |
|------|------|------|
| 대사 포함 여부 | 100% | [PASS/FAIL] |
| 비언어 오디오 설명 | 포함 | [PASS/FAIL/N/A] |
| 화자 식별 | 표시됨 | [PASS/FAIL/N/A] |
| 동기화 정확도 | ±0.5초 | [PASS/WARN] |
| 줄 수/글자 수 | 2줄/21자 | [PASS/WARN] |

## 색상 대비 (Color Contrast)
| 요소 | 전경색 | 배경색 | 대비율 | 기준 | 판정 |
|------|--------|--------|--------|------|------|
| [텍스트 요소] | [색상] | [색상] | [비율]:1 | 4.5:1 | [PASS/FAIL] |

## 깜빡임 (Flicker)
| 항목 | 기준 | 판정 |
|------|------|------|
| 초당 깜빡임 횟수 | < 3회 | [PASS/WARN] |

## 오디오 디스크립션 (Audio Description)
| 항목 | 기준 | 판정 |
|------|------|------|
| 시각 정보 음성 설명 | 제공됨 | [PASS/FAIL/N/A] |

## 전체 판정: [PASS / PASS with warnings / FAIL]
[수정 필요 항목 목록]
```

### Step 6: 수정 가이드

FAIL 항목에 대한 구체적 수정 방법을 제안합니다.

## 다음 단계

접근성 검증 완료 후 `/qc-check` 또는 `/deliver`로 진행합니다.

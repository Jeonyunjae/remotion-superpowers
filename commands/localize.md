---
name: localize
description: Create localized versions of the video — translate subtitles, generate dubbed narration in other languages, and replace on-screen text for international distribution.
---

# Localize — 다국어 영상 제작

마스터 영상에서 다국어 버전을 생성합니다. 자막 번역, TTS 더빙, 화면 내 텍스트 교체를 포함합니다.

## 사전 조건

다음이 존재해야 합니다:
- 렌더링된 마스터 영상
- `docs/04-script.md` (원본 나레이션)
- SRT 자막 파일 (있을 경우)

## 프로세스

### Step 1: 대상 언어 확인

> "어떤 언어로 현지화하나요? (복수 선택 가능)\n1. 영어 (English)\n2. 일본어 (日本語)\n3. 중국어 간체 (简体中文)\n4. 중국어 번체 (繁體中文)\n5. 스페인어 (Español)\n6. 기타"

> "현지화 방식은?\n1. 자막만 (원본 오디오 유지 + 번역 자막)\n2. 더빙 (TTS로 번역 나레이션 생성)\n3. 자막 + 더빙 모두"

### Step 2: 스크립트 번역

원본 스크립트에서 나레이션 텍스트를 추출합니다:

```bash
cat docs/04-script.md
```

**번역 방법:**
- AI 번역으로 초안 생성
- 사용자에게 번역 검수 요청

> "번역 초안을 생성했습니다. 전문 번역이 필요하거나 수정할 부분이 있으면 알려주세요."

### Step 3: 자막 생성

번역된 텍스트로 SRT 파일을 생성합니다. 원본 SRT의 타이밍을 유지합니다.

```bash
mkdir -p delivery/subtitles
```

각 언어별로 `delivery/subtitles/subtitles-[lang].srt` 형식으로 저장합니다.

### Step 4: 더빙 생성 (선택)

`config.yaml`의 TTS 프로바이더를 사용하여 번역된 스크립트로 나레이션을 생성합니다.

**edge-tts 언어별 음성:**
- English: en-US-AriaNeural, en-US-GuyNeural
- Japanese: ja-JP-NanamiNeural, ja-JP-KeitaNeural
- Chinese: zh-CN-XiaoxiaoNeural, zh-CN-YunxiNeural
- Spanish: es-ES-ElviraNeural, es-ES-AlvaroNeural

```bash
# 예: 영어 더빙
edge-tts --text "[translated script]" --voice en-US-AriaNeural --write-media public/audio/voiceover-en.mp3
```

### Step 5: Remotion 컴포지션 (더빙 버전)

언어별 Remotion 컴포지션을 생성합니다:
- 나레이션 오디오를 언어별 파일로 교체
- 화면 내 텍스트를 번역된 텍스트로 교체
- 자막 컴포넌트의 텍스트를 번역 버전으로 교체

### Step 6: 렌더링

```bash
npx remotion render [CompositionId]-en out/video-en.mp4
npx remotion render [CompositionId]-ja out/video-ja.mp4
```

### Step 7: 결과 정리

```
🌐 Localization Complete

| 언어 | 자막 | 더빙 | 파일 |
|------|------|------|------|
| 한국어 (원본) | ✅ | ✅ | out/video.mp4 |
| English | ✅ | ✅ | out/video-en.mp4 |
| 日本語 | ✅ | ✅ | out/video-ja.mp4 |

자막 파일: delivery/subtitles/
```

## 다음 단계

현지화 완료 후 `/deliver`에 다국어 파일을 포함합니다.

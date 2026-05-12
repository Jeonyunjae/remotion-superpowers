# Progress Tracking — 진행 상황 자동 갱신 규칙

이 규칙은 **모든 커맨드가 산출물을 저장한 직후** 반드시 따라야 하는 공통 규칙입니다.

## 규칙

**커맨드가 산출물 파일을 생성하거나 갱신한 직후**, 다음을 수행합니다:

### 1. docs/00-progress.md 존재 확인

```bash
test -f docs/00-progress.md
```

존재하지 않으면 이 규칙을 건너뜁니다 (`/setup`이 실행되지 않은 프로젝트).

### 2. 해당 커맨드의 상태를 ✅로 갱신

`docs/00-progress.md`에서 방금 실행한 커맨드의 행을 찾아 `⬜`를 `✅`로 변경합니다.

예시 — `/receive-brief` 실행 후:

변경 전:
```
| ⬜ | /receive-brief | docs/01-client-brief.md | 필수 |
```

변경 후:
```
| ✅ | /receive-brief | docs/01-client-brief.md | 필수 |
```

### 3. 전체 요약 갱신

상단 "전체 요약" 테이블의 다음 값을 재계산합니다:

- **전체 진행률**: ✅인 필수 항목 수 / 전체 필수 항목 수
- **필수 완료**: ✅인 필수 항목 수 / 16
- **선택 완료**: ✅인 선택 항목 수 / 18
- **현재 단계**: ✅가 있는 가장 높은 Phase (해당 Phase의 필수가 모두 ✅이면 다음 Phase)
- **다음 명령어**: 현재 단계에서 ⬜인 첫 번째 필수 항목
- **마지막 업데이트**: 현재 날짜 시간

### 4. 사용자에게 진행 상황 한 줄 보고

산출물 저장 + progress 갱신이 끝나면, 커맨드의 마무리 안내와 함께 현재 진행률을 한 줄로 보여줍니다:

> "[커맨드 완료 안내 메시지]
>
> 📊 진행 상황: 필수 [n/16] 완료 | 다음: /[커맨드명]"

## 적용 대상

이 규칙은 다음 커맨드들이 산출물을 저장할 때 적용됩니다:

| Phase | 커맨드 | 산출물 |
|-------|--------|--------|
| 1 | /project-scope | docs/00-project-scope.md |
| 1 | /brand-kit | docs/00-brand-guidelines.md |
| 1 | /select-models | config.yaml |
| 2 | /receive-brief | docs/01-client-brief.md |
| 2 | /creative-brief | docs/02-creative-brief.md |
| 2 | /concept-options | docs/03-concepts.md |
| 2 | /treatment | docs/03-treatment.md |
| 2 | /write-script | docs/04-script.md |
| 2 | /fact-check | docs/04-fact-check.md |
| 2 | /storyboard | docs/05-storyboard.md |
| 2 | /style-frame | docs/style-frames/ |
| 2 | /animatic | src/Animatic.tsx |
| 2 | /prompt-sheet | docs/06-prompt-sheet.md |
| 2 | /legal-check | docs/07-legal-checklist.md |
| 3 | /create-video | out/video.mp4 |
| 3 | /add-voiceover | public/audio/voiceover.mp3 |
| 3 | /add-music | public/audio/music.wav |
| 3 | /generate-image | public/images/ |
| 3 | /generate-clip | public/footage/ |
| 3 | /find-footage | public/footage/ |
| 3 | /add-captions | 자막 컴포넌트 |
| 3 | /audio-mix | 오디오 최적화 |
| 3 | /color-grade | 컬러 보정 |
| 4 | /review-video | 리뷰 피드백 |
| 4 | /qc-check | QC 리포트 |
| 4 | /revision-log | docs/08-revision-log.md |
| 4 | /accessibility | 접근성 리포트 |
| 4 | /thumbnail | out/thumbnails/ |
| 4 | /seo-metadata | docs/10-seo-metadata.md |
| 5 | /export-multi | 플랫폼별 MP4 |
| 5 | /deliver | docs/09-delivery-package.md |
| 5 | /localize | 다국어 MP4 + SRT |
| 6 | /repurpose | out/repurposed/ |
| 6 | /archive | docs/99-archive.md |

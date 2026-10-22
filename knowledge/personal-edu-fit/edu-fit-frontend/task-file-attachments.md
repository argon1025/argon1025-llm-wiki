---
description: 요청·댓글 첨부 파일을 업로드하거나 내려받는 화면을 다룰 때
---

# 탐구 업무 파일 첨부

## 규칙

- 파일 업로드는 fetch가 아니라 XMLHttpRequest 기반 apiUpload로 올림 — fetch는 업로드 진행률을 주지 않음
- 파일 선택 즉시 업로드하는 훅(useUploads)은 20MB(20×1024×1024 bytes) 초과 파일을 요청 없이 용량 초과 실패(tooLarge)로 처리함
- 업로드 용량 초과 판정(isFileTooLarge)은 responseCode가 아니라 HTTP status 413으로 하고 사전 차단과 같은 실패로 매핑함 — 프록시가 envelope 없이 413을 줄 수 있음
- 파일을 받기 직전 그 파일이 실린 응답을 다시 조회해 새 downloadUrl로 window.location.assign함 — 서명 다운로드 경로가 10분 뒤 만료됨
- 받기 직전 다시 조회하는 응답은 요청 첨부·산출물·후속 탐구 파일이면 상세, 댓글 첨부면 댓글 목록임

## 함정

- 업로드 훅(useUploads)에서 업로드 중 제거한 항목은 요청을 중단하지 않고 늦은 응답만 버리며 서버에 남는 고아 파일은 정리하지 않음(의도된 동작)
- 다운로드 요청이 서명 403을 받으면 브라우저가 JSON 응답 화면으로 이동함(의도된 동작) — 다운로드가 페이지 이동(window.location.assign)이라 오류 응답도 브라우저가 그대로 엶

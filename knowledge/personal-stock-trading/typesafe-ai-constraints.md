---
description: TypeSafe AI API 연동 시 오류 처리·요청 한도를 다룰 때
type: external
---

# TypeSafe AI 제약

## 오류 응답

- 오류 응답 본문은 `{detail:{error_type, message}}` 형태 — 401은 error_type: authentication_error, 공식 문서에 명시되어 있지 않음

## 한도와 언어

- rate limit은 250k tokens/s, 1,200 req/min이나 공지 없이 동적으로 바뀜
- 영어 외 언어(한국어 포함)는 정확도가 낮을 수 있음

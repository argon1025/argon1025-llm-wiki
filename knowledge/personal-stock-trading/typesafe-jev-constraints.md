---
description: TypeSafe AI Jev API 응답 형태나 한도를 다룰 때
type: external
---

# TypeSafe Jev API 제약

## 한도

- rate limit(250k tokens/s, 1,200 req/min)은 공지 없이 동적으로 바뀜
- 영어 외 언어(한국어 포함)는 정확도가 낮을 수 있어 질문 문구는 영어 우선으로 검토해야 함

## 오류 응답 형태

- 오류 응답 본문은 `{ detail: { error_type, message } }` 형태 — 공식 문서에 명시되어 있지 않음
- 401은 `error_type: 'authentication_error'`

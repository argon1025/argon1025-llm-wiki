---
description: Jev 평가 요청의 모델·질문 작성·오류·재시도를 다룰 때
---

# TypeSafe AI Jev 연동

## 규칙

- Jev 평가 요청의 모델(model)은 별칭 `jev-latest`를 쓰지 않고 환경 변수 `TYPESAFE_AI_JEV_MODEL`의 고정 버전(예: `jev-1.13.0`)으로 지정함 — 별칭은 새 릴리스 시 답이 바뀌어 같은 입력 재판정 요건을 깨뜨림
- TypeSafe AI 연동 계층(TypeSafeAiHttpClient)은 429·529를 재시도하지 않고 TypeSafeAiException만 던짐 — 재시도 여부는 Jev 로그 등 상위 계층이 정함
- 상위 계층의 재시도·분류는 TypeSafeAiException.status를 우선 기준으로 삼음 — 오류 응답 본문 모양을 공식 문서가 명시하지 않음
- Jev의 state·질문 문구는 영어 우선으로 검토함 — 영어 외 언어(한국어 포함)는 정확도가 낮을 수 있음

## 함정

- TypeSafe AI 오류 응답 본문은 `{ detail: { error_type, message } }` 형태이고 401은 error_type이 `authentication_error`임 — 공식 문서에 없는 모양
- Jev rate limit(250k tokens/s, 1,200 req/min)은 공지 없이 동적으로 바뀜
- questions를 변수로 빼서 TypeSafeAiSystemOneApi.evaluate()에 넘기면 `as const`를 붙임(`satisfies TypeSafeAiQuestions` 병용 가능) — answers 타입 추론이 const 타입 파라미터에 기대어, 없으면 type이 string으로 넓어져 컴파일 오류가 남

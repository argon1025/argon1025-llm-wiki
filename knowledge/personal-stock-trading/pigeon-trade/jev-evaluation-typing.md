---
description: TypeSafeAi 평가 응답·질문 타입을 다룰 때
type: convention
---

# TypeSafeAi 평가 타이핑

## 규칙

- Jev 평가 응답 타입을 질문별 제네릭 매핑 대신 단순 Record<string, 답변 타입>으로 두는 안은 기각됨 — 상위 계층의 내로잉 부담
- 평가 호출(TypeSafeAiSystemOneApi.evaluate())에 questions를 변수로 빼서 넘길 때 as const(또는 satisfies와 as const 병용)가 없으면 choice 답변 타입이 좁아지지 않고 오히려 type: string이 TypeSafeAiQuestion에 맞지 않아 컴파일 오류가 남

---
description: API 엔드포인트나 DTO의 Swagger 문서화를 작성하거나 응답 코드를 표기할 때
type: convention
---

# Swagger API 문서화 규약

## DTO 필드 문서화

- 필드마다 `@ApiProperty({ description, example })`를 명시하고 필드 JSDoc(설명·`@example`)은 두지 않음 — `@nestjs/swagger` CLI 플러그인안은 기각, 설명 출처를 데코레이터 하나로 둠

## 응답 문서화

- 서버 오류 응답(INTERNAL_SERVER_ERROR, 500)은 엔드포인트별로 표기하지 않고 `swagger.setup.ts` 문서 설명에 안내 — 모든 API 공통

## enum 배열 동기화

- 시장·통화(BrokerageMarket·BrokerageCurrency)의 응답 DTO Swagger enum은 satisfies 배열로 따로 적음 — 런타임 값이 없는 타입 유니언
- 시장·통화 값이 늘면 응답 DTO 2종의 enum 배열을 함께 갱신 — 타입 검사가 누락을 잡지 못함

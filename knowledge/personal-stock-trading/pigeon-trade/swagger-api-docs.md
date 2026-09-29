---
description: Swagger API 문서의 DTO 필드나 실패 응답을 작성·검증할 때
---

# Swagger API 문서

## 규칙

- DTO 필드의 Swagger 설명·예시는 `@nestjs/swagger` CLI 플러그인 없이 필드마다 `@ApiProperty({ description, example })`로 명시하고 필드 JSDoc 설명은 두지 않음 — 설명 출처를 데코레이터 하나로 유지함
- 엔드포인트별 실패 응답(`ApiErrorResponse`)에는 500 `INTERNAL_SERVER_ERROR`를 표기하지 않고 Swagger 문서 설명(`setupSwagger`)에서 한 번만 안내함 — 모든 API에 공통인 응답임
- Swagger 문서는 자동 테스트 없이 앱 기동 후 수동으로 검증함 — vitest는 `emitDecoratorMetadata`를 내보내지 않아 테스트에서 `@ApiProperty` 타입 추론이 동작하지 않음

## 함정

- 공용 타입 시장(`Market`)·통화(`Currency`)에 값을 더하면 Swagger enum에 따로 적은 `satisfies Market[]`·`satisfies Currency[]` 배열을 모두 찾아 함께 갱신함 — 런타임 값 없는 타입 유니언이라 배열에 값이 빠져도 타입 검사가 잡지 못함

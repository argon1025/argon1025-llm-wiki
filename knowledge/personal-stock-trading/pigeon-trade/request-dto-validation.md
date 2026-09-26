---
description: 쿼리 파라미터 DTO에 숫자 타입 변환이 필요할 때
type: convention
---

# 쿼리 DTO 숫자 변환

## 규칙

- 숫자 쿼리 파라미터는 DTO에 @Type(() => Number)를 붙여야 숫자로 변환됨 — 전역 ValidationPipe에 enableImplicitConversion이 없음(src/common/http/validation-pipe.factory.ts)

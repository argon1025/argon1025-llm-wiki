---
description: 서비스 예외를 응답 코드와 함께 발행할 때
type: convention
---

# 서비스 예외 발행

## 발행

- 서비스 예외는 Nest 내장 예외(NotFoundException 등)에 응답 코드 목록 항목(ResponseCode.X, code·message 객체)을 넘겨 발행함 — 문자열 코드 전달은 오타 위험으로 기각
- HTTP status는 응답 코드 목록에 두지 않고 던지는 쪽의 예외 클래스로 명시함
- 검증 사유처럼 message만 바꿀 때는 항목을 펼친 뒤 message를 덮어씀

```ts
throw new BadRequestException({ ...ResponseCode.INVALID_REQUEST, message: '...' });
```

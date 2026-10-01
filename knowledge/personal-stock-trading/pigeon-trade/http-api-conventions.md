---
description: HTTP API 응답 DTO·예외·쿼리 변환 규약을 따라 엔드포인트를 추가할 때
---

# HTTP API 규약

## 규칙

- 응답 DTO는 필드가 같아도 향후 요구가 갈릴 수 있으면 엔드포인트별로 분리함
- 응답 DTO 변환은 컨트롤러에서 plainToInstance를 명시 호출하고 @SerializeOptions는 쓰지 않음
- 숫자 쿼리 파라미터는 DTO에 `@Type(() => Number)`를 붙임 — 전역 ValidationPipe는 transform: true이나 enableImplicitConversion이 없어 붙이지 않으면 숫자로 변환되지 않음
- 응답 DTO의 luxon DateTime 필드에는 `@Type(() => String)`을 붙이고 @Transform은 value 대신 원본 obj의 값을 변환함 — plainToInstance가 타입 지정 없는 객체 값을 new DateTime()으로 복제하다 실패함

## 결정

- 서비스 예외에 ResponseCode 항목 객체 대신 문자열 코드를 넘기는 방식은 기각함 — 오타 위험

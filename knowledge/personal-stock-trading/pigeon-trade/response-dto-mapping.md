---
description: API 응답 DTO를 정의하거나 컨트롤러 반환을 변환할 때
type: convention
---

# 응답 DTO 구성

## 구성

- 응답 DTO는 필드가 같아도 엔드포인트별로 분리함 — 엔드포인트별 요구사항이 이후 갈릴 수 있음
- 컨트롤러가 서비스 결과를 plainToInstance로 응답 DTO 인스턴스로 변환해 반환하고 @SerializeOptions는 쓰지 않음

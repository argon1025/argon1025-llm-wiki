---
description: API 응답 DTO를 정의하거나 컨트롤러 반환을 변환할 때
type: convention
---

# 응답 DTO 구성

## 구성

- 응답 DTO는 필드가 같아도 엔드포인트별로 분리함 — 엔드포인트별 요구사항이 이후 갈릴 수 있음
- 컨트롤러가 서비스 결과를 plainToInstance로 응답 DTO 인스턴스로 변환해 반환하고 @SerializeOptions는 쓰지 않음

## decimal 응답 형식

- 가격·거래량 등 decimal(24,8) 응답은 JSON number 대신 Decimal#toFixed() 문자열로 내보냄 — 정밀도 손실 없이 에이전트·MCP 소비에도 안전함
- oxlint 규칙 unicorn(require-number-to-fixed-digits-argument)는 decimal.js Decimal#toFixed()도 Number로 오인해 경고함
- 응답 DTO의 Decimal#toFixed()는 이 경고를 그대로 두고 인자 없이 호출함 — 인자를 주면 소수 자릿수가 고정되어 값이 바뀜

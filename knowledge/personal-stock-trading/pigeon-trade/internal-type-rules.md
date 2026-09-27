---
description: 내부 계층 타입을 명명하거나 시각 값 타입을 정할 때
type: convention
---

# 내부 타입 규칙

## 타입 명명

- 외부 연동 계층(src/integration/)과 HTTP 경계 DTO(src/{도메인}/dto/) 밖 내부 계층의 입력·출력 타입은 Request·Response 대신 Option·Result로 명명함

## 시각 타입

- 애플리케이션 코드의 시각 타입은 JS Date 대신 luxon DateTime으로 통일함 — 곧 필요한 거래소 시간대 계산을 위해 나중 전환보다 지금 통일하는 비용이 작다고 판단
- 시각 값은 내부 계산·DB 저장·API 응답 모두 UTC(응답은 UTC ISO 8601 문자열)로 통일하고, 증권사 응답 오프셋은 증권사 구현체에서 UTC로 변환함

## 공용 값 타입 범위

- 여러 도메인이 공유하는 값 타입(통화 Currency·상장 시장 Market)만 src/common/type/에 두고 엔티티 교차 참조·응답 DTO·Result 타입은 소유 도메인에 유지함
- 외부 연동 계층(src/integration/**)은 공용 타입(src/common/type/)을 참조하지 않고 자체 타입(TossInvestCurrency·TossInvestMarket)을 유지하며 증권사 추상 계층(src/brokerage/)은 참조를 허용함 — 연동 계층은 외부 사양 변경에 따라 바뀜

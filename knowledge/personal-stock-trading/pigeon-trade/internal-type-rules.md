---
description: 내부 계층 타입을 명명하거나 시각 값 타입을 정할 때
type: convention
---

# 내부 타입 규칙

## 타입 명명

- 외부 연동 계층(src/integration/) 밖 내부 계층의 입력·출력 타입은 Request·Response 대신 Option·Result로 명명함

## 시각 타입

- 애플리케이션 코드의 시각 타입은 JS Date 대신 luxon DateTime으로 통일함 — 곧 필요한 거래소 시간대 계산을 위해 나중 전환보다 지금 통일하는 비용이 작다고 판단

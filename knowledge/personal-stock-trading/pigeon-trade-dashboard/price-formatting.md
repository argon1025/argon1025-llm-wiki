---
description: 종목 가격·지갑 금액·체결 시각의 표시 형식을 다룰 때
---

# 가격·금액·시각 표시 형식

## 규칙

- 금액 표시(formatPrice·formatMoney)는 KRW·USD 모두 통화 코드 대신 기호(₩·$)를 앞에 붙임
- 지갑 금액은 formatPrice가 아닌 formatMoney로 표시함 — formatPrice의 Number 변환은 decimal(24,8) 정밀도를 잃고 formatMoney는 문자열을 Intl에 그대로 넘겨 소수 8자리까지 반올림 없이 보존함
- 지갑 화면의 주문 체결 시각·트레이드 기간(formatDateTime)은 종목 상세 차트(toChartTime)와 달리 종목 시간대가 아닌 브라우저 현지 시각(KST)으로 표시함(의도된 동작)

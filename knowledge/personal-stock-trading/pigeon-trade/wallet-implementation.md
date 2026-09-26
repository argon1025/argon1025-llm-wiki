---
description: 지갑 트레이드 정렬이나 종목 시장·통화 문자열 처리를 다룰 때
type: convention
---

# 지갑 구현 방식

## 트레이드 정렬

- 트레이드 정렬은 시작 시각(opened_at) 대신 구간 첫 주문 id 내림차순으로 함 — 체결 시각(created_at)이 소수 초 없는 datetime이라 같은 초에 열린 트레이드끼리 순서가 갈리지 않으며, 주문 id는 체결 순서와 같아 시작 시각 내림차순과 결과가 같음

## 통화 처리

- 지갑은 종목 시장·통화(stock.market·stock.currency)를 좁은 타입 대신 string으로 다룸 — stock 도메인은 수정하지 않기로 해 두 컬럼이 $type 없는 string 컬럼으로 남음

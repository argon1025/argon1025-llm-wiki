---
description: 지갑 트레이드 구간·손익을 계산하거나 OPEN·CLOSED 상태를 다룰 때
---

# 지갑 트레이드

## 적용 대상

- 트레이드를 계산하는 pigeon-trade의 pigeon-wallet 모듈, 트레이드 상태를 표시하는 pigeon-trade-dashboard

## 규칙

- 지갑 트레이드(findTrades)는 지갑 주문 이력을 종목별로 보유 0에서 첫 매수 체결(openedAt)로 열고 보유 0으로 돌아오는 매도로 닫은 구간임

## 코드값

원본: `PigeonWalletTradeStatus` 전 2종

| 코드 | 뜻 |
|---|---|
| `OPEN` | 종목 보유 수량이 0이 아닌 진행 중 구간 — 손익(pnl)과 청산 시각(closedAt)이 null |
| `CLOSED` | 보유 수량이 0으로 돌아오는 매도로 청산된 구간 — 손익(pnl)은 구간 현금 증감(cash_delta) 합, 청산 시각(closedAt)은 청산 매도 체결 시각 |

## 함정

- 평균 단가(average_cost)가 나누어떨어지지 않으면 CLOSED 트레이드 손익(pnl)과 매도 주문 realized_pnl 합이 소수 8자리 이하에서 어긋나며 정상임, 정본은 구간 cash_delta 합 — 평균 단가를 decimal(24,8)로 반올림 저장함

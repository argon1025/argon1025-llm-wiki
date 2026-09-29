---
description: 지갑 주문 체결가·수수료·제세금, 입금·잔고·포지션 조회 API를 다룰 때
---

# 지갑 주문

## 적용 대상

- 주문·잔고·포지션을 처리하는 pigeon-trade의 pigeon-wallet 모듈, 그 API를 호출해 지갑 화면을 그리는 pigeon-trade-dashboard

## 규칙

- 지갑 주문(placeOrder)은 요청에 가격을 받지 않고 저장된 해당 종목 최신 1m 봉 종가로 슬리피지 없이 즉시 전량 체결함
- 최신 1m 봉이 오래돼도(장 마감·동기화 중단) 그 종가로 체결하고 기준 봉 시각(priceTimestamp)만 주문에 남김(의도된 동작)
- 종목에 1m 봉이 없으면 주문은 400 PRICE_NOT_AVAILABLE로 거부됨
- 지갑 수수료 요율(MARKET_FEE_RATES)은 토스증권 요율로 국내 0.015%(KRX 기준, NXT 0.014%는 미적용)·미국 0.1%임
- 매도 제세금은 KOSPI·KOSDAQ 0.20%(거래세 0.05%+농특세 0.15%)이고 미국은 SEC fee 0.00206%(2026-04-02 체결분부터, 최소 $0.01), FINRA TAF는 미적용
- 국내 수수료·제세금은 원 미만 절사, 미국 수수료는 센트 반올림, SEC fee는 센트 올림
- 지갑은 통화별 현금을 분리하고 환전이 없음
- 입금액(DepositRequest.amount)은 0 초과, 정수부 16자리·소수부 8자리 이내 문자열(`^(?=.*[1-9])\d{1,16}(\.\d{1,8})?$`)만 받고 위반 시 400 INVALID_REQUEST — 저장 컬럼 decimal(24,8)에 맞춤
- 포지션 평균 단가(averageCost)는 매수 수수료를 포함한 단가이고 매도는 평균 단가를 바꾸지 않음
- 지갑 상세 조회의 미실현 손익(unrealizedPnl)은 (최신 1m 종가 − 평균 단가) × 수량이며 매도 시 수수료·제세금을 차감하지 않음
- 최신 1m 봉이 없으면 포지션 평가 필드가 null이고 pigeon-trade-dashboard는 그 포지션을 평가 불가로 표시함
- 지갑 상세 조회(GetWalletResponse)는 balances에 입금한 통화만, positions에 보유 수량 1 이상 종목만 담음
- pigeon-trade 지갑 API에는 지갑 삭제·이름 변경이 없음
- 지갑 목록은 id 오름차순, 주문 이력은 id 내림차순, 트레이드는 최근에 열린 트레이드 먼저이며 세 목록 모두 페이지네이션 없이 전체를 반환함
- 지갑 목록(GET /api/wallets) 응답은 id·name·createdAt만 담아 잔고를 보려면 pigeon-trade-dashboard가 지갑마다 상세를 조회해야 함
- 주문 이력·트레이드 응답에는 종목명이 없어 종목 목록 응답에서 symbol로 찾아야 하며, 종목명(name)은 지갑 상세 포지션에만 담김
- 숫자가 아닌 walletId는 400 INVALID_REQUEST로 거부되며 pigeon-trade-dashboard는 이 응답과 404 WALLET_NOT_FOUND를 모두 존재하지 않는 지갑으로 표시함

## 코드값

원본: `PigeonWalletOrderSide` 전 2종

| 코드 | 뜻 | 현금 증감(cashDelta) | 제세금(tax) | 실현 손익(realizedPnl) |
|---|---|---|---|---|
| `BUY` | 매수 | 음수(체결 금액+수수료) | 0 | null |
| `SELL` | 매도 | 양수(체결 금액−수수료−제세금) | 매도 제세금 적용 | (체결가−평균 매입 단가)×수량−수수료−제세금 |

## 함정

- KR_ETC 시장 매도는 ETF 등 거래세 면제 상품도 국내 주식 제세금 0.20%로 계산해 비용이 과대 계상될 수 있음 — 면제 상품을 구분할 정보가 없음

## 결정

- 주문 요청에 가격을 받는 안은 버림 — 도메인 격리는 완전하나 에이전트가 가격을 임의로 넣을 수 있음

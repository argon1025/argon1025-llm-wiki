---
description: 1분봉 저장 이벤트 페이로드를 바꾸거나 구독 리스너를 추가할 때
---

# 1분봉 저장 이벤트

## 규칙

- 종목 현재가(price·priceTimestamp) 구독·저장은 stock 도메인 리스너(StockPriceListener)가 맡고 market-data는 1m 저장 이벤트(Candle1mSavedEvent) 페이로드에 봉 값만 실음 — stock 테이블 소유 도메인이 stock임
- 30m·1h 봉 기준 구독자는 1m 저장 이벤트 대신 집계 봉 저장 이벤트(CANDLE_AGGREGATED_SAVED_EVENT)를 구독함 — 1m 저장 이벤트 구독자(집계·현재가)는 서로 기다리지 않고 동시에 실행돼 같은 틱 집계 반영 전 값을 읽을 수 있음

## 결정

- stock 리스너가 MarketDataExternalService로 최신 1m을 조회하는 안은 기각함 — market-data가 이미 stock 엔티티에 의존해 두 모듈이 서로 의존하고 이벤트마다 조회가 늘어남

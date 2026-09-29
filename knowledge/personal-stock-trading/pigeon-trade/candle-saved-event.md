---
description: 1분봉 저장 이벤트 페이로드를 바꾸거나 구독 리스너를 추가할 때
---

# 1분봉 저장 이벤트

## 규칙

- 종목 현재가(price·priceTimestamp) 구독·저장은 stock 도메인 리스너(StockPriceListener)가 맡고 market-data는 1m 저장 이벤트(Candle1mSavedEvent) 페이로드에 봉 값만 실음 — stock 테이블 소유 도메인이 stock임

## 결정

- stock 리스너가 MarketDataExternalService로 최신 1m을 조회하는 안은 기각함 — market-data가 이미 stock 엔티티에 의존해 두 모듈이 서로 의존하고 이벤트마다 조회가 늘어남

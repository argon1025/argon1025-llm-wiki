---
description: 종목 국면 판정 창·설명 문구·리스너를 바꾸거나 MCP 국면 제외를 다룰 때
---

# 종목 국면 판정 구현

## 적용 대상

- pigeon-trade의 stock 모듈(국면 판정 서비스·리스너)과 agent-mcp 모듈(`list_stocks` 응답)

## 규칙

- 국면 판정 창·입력을 바꿀 때는 토스 1m 과거 조회를 저장과 같은 UTC 버킷으로 30m 집계해 창별로 Jev를 실호출해 비교함 — 저장 봉과 같은 입력으로 비교해야 운영 판정과 일치함
- 창 비교의 주 근거는 SOXL·SOXS 거울상 일치율과 반복 호출 안정성이며 12시간 뒤 방향 적중 지표는 주 근거로 쓰지 않음 — 12시간 뒤 방향 적중은 짧은 창에 유리하게 치우침
- 창 비교 결과는 확정 근거로 보지 않음 — 비교 표본(반도체 연관 6종목 1개월)의 독립성이 낮음
- 국면 설명 문구(`stock.regime` DB 코멘트·`GET /api/stocks` Swagger description·`StockResult.regime` JSDoc)에는 판정 봉 수를 적지 않고 봉 수는 `REGIME_BARS` JSDoc에만 둠 — 창 길이를 바꿀 때 마이그레이션이 필요 없게 함

## 함정

- `GetStocksResponse`에 regime으로 시작하는 필드를 더하면 MCP `list_stocks` 응답에서도 함께 빠짐 — `list_stocks`가 공유 응답 DTO를 `plainToInstance`의 `excludePrefixes: ['regime']`로 걸러 국면 필드를 뺌
- 국면 리스너(`StockRegimeListener`)는 집계 리스너(`CandleAggregationListener`)와 달리 실행 중 건너뛰기(running 집합)를 두지 않음(의도된 동작) — 같은 종목 판정은 30분 간격이고 겹쳐도 각 결과가 유효한 최신 판정임

---
description: 1m 저장 이벤트로 30m·1h 봉 집계를 고치거나 워터마크를 다룰 때
type: convention
---

# 캔들 이벤트 기반 30m·1h 집계

## 워터마크 집계

- 이미 저장된 30m·1h 봉은 겹친 1m 가격이 뒤늦게 바뀌어도 워터마크(봉 단위별로 저장된 해당 파생 봉의 최신 봉 종료 시각) 뒤라 다시 집계되지 않음 — 의도적 단순화이며 재집계 수단은 아직 정해지지 않음(src/market-data/service/candle-aggregation.service.ts)
- 1m 1회 읽기 한도(AGGREGATION_SOURCE_LIMIT 20000행, 약 28거래일) 안에 마감 버킷이 하나도 없으면 워터마크가 전진하지 않아 집계가 멈춤(src/market-data/service/candle-aggregation.service.ts#aggregate)

## 집계 테스트 첫 버킷

- 집계 결과를 검증하는 테스트는 첫 정규장 1m 앞에 정규장 밖 1m(예: 장 시작 전 08:00)을 하나 두어야 첫 버킷이 저장됨 — 저장된 30m·1h가 없으면 1m 최오래된 봉이 속한 버킷을 부분 봉으로 버림(test/e2e/market-data/service/candle-aggregation.service.e2e-spec.ts)

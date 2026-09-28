---
description: 1m 저장 이벤트로 30m·1h 봉 집계를 고치거나 워터마크를 다룰 때
type: convention
---

# 캔들 이벤트 기반 30m·1h 집계

## 정각 버킷 시간대 전제

- 30m·1h 버킷은 UTC epoch 기준 정각으로 나눠 정시 오프셋 시장(KST +09:00, 뉴욕 −04:00/−05:00)에서만 거래소 현지 정각과 같음 — 30분 오프셋 시장을 추가하면 버킷 계산을 거래소 시간대 기준으로 바꿔야 함(src/market-data/service/candle-aggregation.service.ts#toBucketRange)

## 워터마크 집계

- 이미 저장된 30m·1h 봉은 겹친 1m 가격이 뒤늦게 바뀌어도 워터마크(봉 단위별로 저장된 해당 파생 봉의 최신 봉 종료 시각) 뒤라 다시 집계되지 않음 — 의도적 단순화(src/market-data/service/candle-aggregation.service.ts)
- 집계 규칙이 바뀌면 저장된 30m·1h를 마이그레이션에서 삭제해 재집계함 — 워터마크가 사라져 종목별 다음 1m 저장 이벤트가 가장 오래된 1m부터 다시 집계하므로 별도 재집계 코드가 없음(migrations/Migration20260928060221.ts)
- 1m 1회 읽기 한도(AGGREGATION_SOURCE_LIMIT 20000행, 약 28거래일) 안에 마감 버킷이 하나도 없으면 워터마크가 전진하지 않아 집계가 멈춤(src/market-data/service/candle-aggregation.service.ts#aggregate)

## 재집계 공백

- 30m·1h 삭제 후 재집계를 종목별 다음 1m 저장 이벤트에 맡겨 주말·휴장 배포면 다음 거래 시작까지, 동기화 비활성 종목은 다시 켤 때까지 30m·1h가 비어 있음 — 의도적 단순화이며 문제가 되면 기동 시 전 종목 집계 트리거를 추가(migrations/Migration20260928060221.ts)

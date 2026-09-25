---
description: 시세 동기화 Cron에 시세를 더하거나 백필 예산·요청 한도를 다룰 때
type: convention
---

# 시세 동기화 구성

## 도메인 경계

- 시세 도메인(src/market-data/)은 증권사 중립 계약(src/brokerage/의 brokerage.constants.ts·brokerage.exception.ts·interface/)과 src/stock/entity/만 import하고 src/integration/·src/brokerage/strategy/를 import하지 않음 — 토스 스펙이 내부 서비스 간 규약에 영향을 주지 않도록

## 요청 한도와 페이지 크기

- 시세 동기화는 요청 한도 초과(429)를 받으면 틱 안에서 대기·재시도하지 않고 틱을 종료함 — Retry-After 기반 재시도와 연동 예외의 응답 헤더 보존은 다음 틱이 DB 기준으로 이어받아 불필요해 기각

## 백필

- 캔들 백필의 봉단위 종료는 페이지 최오래된 봉이 수집 하한 이하이거나 빈 페이지일 때로 판정함 — 받은 봉 수가 count보다 작으면 끝으로 보는 안은 증권사가 항상 꽉 찬 페이지를 준다는 가정이라 기각
- 백필 예산(CANDLE_BACKFILL_PAGES_PER_TICK)은 종목 수×예산×2 호출이 1분 틱 안에 순차 초당 약 10회로 끝나는 범위(종목 10개면 25 이하)로 둠 — 넘으면 틱이 겹쳐 다음 틱이 건너뛰어짐
- 종목의 백필 요청(backfillEnabled)을 다시 켜면 DB 최오래된 봉에서 확인 호출 1회 후 바로 false로 돌아가므로 과거 봉 재수집은 해당 봉 삭제가 선행돼야 함

## 동시 실행 방지 한계

- 동시 실행 방지가 프로세스 내 플래그라 여러 worktree가 같은 DB로 동시에 기동하면 증권사 호출이 중복되고 레이트 리밋을 나눠 씀 — 데이터는 upsert라 안전하며 다중 인스턴스 운영 시 MySQL GET_LOCK 도입 필요

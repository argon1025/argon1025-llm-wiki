---
description: 시세 동기화 Cron의 요청 한도나 비활성 종목 재개를 다룰 때
type: convention
---

# 시세 동기화 구성

## 도메인 경계

- 시세 도메인(src/market-data/)은 증권사 중립 계약(src/brokerage/의 brokerage.constants.ts·brokerage.exception.ts·interface/)과 src/stock/entity/만 import하고 src/integration/·src/brokerage/strategy/를 import하지 않음 — 토스 스펙이 내부 서비스 간 규약에 영향을 주지 않도록

## 요청 한도와 페이지 크기

- 시세 동기화는 요청 한도 초과(429)를 받으면 틱 안에서 대기·재시도하지 않고 틱을 종료함 — Retry-After 기반 재시도와 연동 예외의 응답 헤더 보존은 다음 틱이 DB 기준으로 이어받아 불필요해 기각

## 장기 비활성 복구

- 종목의 시세 동기화(enabled)를 오래 끈 뒤 켜면 그 기간 전체를 한 틱에 역방향으로 모아 저장하며, 한 틱이 1분을 넘으면 다음 틱이 건너뛰어지므로 문제가 되면 해당 종목 1m을 지우고 신규처럼 시작함

## 동시 실행 방지 한계

- 동시 실행 방지가 프로세스 내 플래그라 여러 worktree가 같은 DB로 동시에 기동하면 증권사 호출이 중복되고 레이트 리밋을 나눠 씀 — 데이터는 upsert라 안전하며 다중 인스턴스 운영 시 MySQL GET_LOCK 도입 필요

---
description: MikroORM 엔티티 컬럼·FK를 정의하거나 EntityManager로 조회·upsert할 때
---

# MikroORM 엔티티

## 규칙

- 애플리케이션 시각 타입은 JS Date 대신 luxon DateTime으로 통일함 — 거래소 시간대 계산이 곧 필요해 나중에 전환하는 비용보다 지금 통일하는 비용이 작음
- luxon 엔티티 필드는 LuxonDateTimeType으로 UTC SQL 문자열을 직접 읽고 쓰며 p.datetime()을 쓰지 않음 — MikroORM MySQL driver가 dateStrings: true로 datetime을 문자열로 넘김
- manyToOne 두 개로만 된 복합 PK 엔티티(pigeon_wallet_position)는 두 FK에 deleteRule·updateRule을 no action으로 명시함 — MikroORM 7이 이를 조인 테이블로 보고 cascade를 붙여 종목·지갑 삭제가 보유 포지션을 조용히 지움
- HTTP 요청 밖(Cron·@OnEvent 리스너)에서는 주입된 EntityManager 대신 em.fork()로 만든 EM을 씀 — allowGlobalContext가 꺼져 있어 주입된 EM으로 조회하면 전역 컨텍스트 오류가 남
- HTTP 요청 경로와 요청 밖(Cron·@OnEvent) 양쪽에서 불리는 서비스 메서드(CandleService·TossInvestSymbolService·TossInvestAccessTokenService)는 요청 경로에서도 항상 메서드 안에서 `this.em.fork()`로 EM을 만듦 — 주입 EM은 전역 orm.em이라 요청 밖에서 그대로 쓰면 `Using global EntityManager instance methods for context specific actions is disallowed`가 남
- nullable manyToOne 참조는 instanceof 대신 `?.id ?? null`로 좁힘 — defineEntity로 정의한 엔티티는 클래스가 아닌 상수라 instanceof를 쓸 수 없음
- em.find에 exclude를 준 결과도 받는 공용 변환 메서드(AgentDecisionService.toResult)는 필요한 속성만 Pick으로 받음 — exclude: ['context']를 주면 Loaded 타입에서 context가 빠짐

## 함정

- 소수 초 없는 datetime 컬럼은 저장 시 밀리초를 버리지 않고 반올림함 — 응답 createdAt 08:30:01.615가 이후 조회에서 08:30:02.000으로 나옴
- MikroORM 7 upsertMany는 첫 행에 없는 키의 컬럼을 어느 행에서도 갱신하지 않아 기존 값이 남음 — 갱신 컬럼 목록을 첫 행의 키로 정함
- 요청 밖 컨텍스트를 만들 때는 @mikro-orm/nestjs README가 안내하는 @CreateRequestContext() 대신 @mikro-orm/core의 RequestContext.create를 씀 — 설치본 MikroORM(7.2.1)에 @mikro-orm/decorators가 없음

## 결정

- 요청 밖 호출 경로에 RequestContext.create를 두거나 요청 밖 전용 캔들 external provider를 신설하는 대안은 버림 — 기존 fork 방식과 도메인당 ExternalService 위임 1개 규약에서 벗어남

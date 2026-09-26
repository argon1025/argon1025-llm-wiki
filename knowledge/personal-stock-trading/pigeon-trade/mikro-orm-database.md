---
description: MikroORM 엔티티 nullable 속성을 좁히거나 마이그레이션 테이블 collation을 다룰 때
type: convention
---

# MikroORM 엔티티·마이그레이션 함정

## nullable 타입 좁히기

- `defineEntity`의 `.nullable()` 속성은 `T | null | undefined`로 추론되어 `!== null` 비교로는 좁혀지지 않으므로 `typeof`·`instanceof`로 좁힘
- nullable 다대일 참조(manyToOne)는 `?.id ?? null`로 좁힘 — `defineEntity`로 정의한 엔티티는 클래스가 아닌 상수라 `instanceof`로 좁힐 수 없음

## 마이그레이션

- 마이그레이션 테이블 DDL은 `default character set utf8mb4`만 붙고 collation을 지정하지 않아 서버 설정(`utf8mb4_unicode_ci`)과 달리 테이블은 MySQL 8 기본값 `utf8mb4_0900_ai_ci`로 생성됨

## 심볼 collation 의존

- 테이블 collation을 바꾸면 종목 심볼(stock.symbol) 대소문자 처리를 코드로 옮겨야 함 — unique 인덱스와 조회의 대소문자 무시가 코드가 아니라 collation(`utf8mb4_0900_ai_ci`)에 의존함

## datetime 소수 초

- 소수 초 없는 datetime 컬럼(stock.created_at·agent_decision.created_at)은 저장 직후 반환한 엔티티 값은 밀리초를 담고 저장 값은 밀리초를 초 단위로 반올림해, 등록 응답과 이후 조회의 시각이 다름

## exclude 조회 타입

- 제외 조회와 전체 조회가 함께 쓰는 변환 메서드는 필요한 속성만 `Pick`으로 받음 — `em.find`에 `exclude`를 주면 반환 타입의 `Loaded`에서 그 속성이 빠져 전체 엔티티 타입 인자에 넘길 수 없음

## FK 인덱스

- MikroORM이 만든 FK 단독 인덱스(agent_decision.stock_id)는 복합 인덱스와 중복으로 보여도 지우지 않음 — FK 컬럼이 선두가 아닌 복합 인덱스((wallet_id, stock_id, id))는 FK를 받치지 못함

## 요청 밖 EntityManager

- HTTP 요청 밖(Cron·@OnEvent 리스너)에서는 주입된 EntityManager 대신 `em.fork()`로 만든 EM을 씀 — MikroORM 전역 컨텍스트(allowGlobalContext)가 꺼져 있어 주입된 EM으로 조회하면 전역 컨텍스트 오류

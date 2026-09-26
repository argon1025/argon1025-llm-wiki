---
description: MikroORM 엔티티 nullable 속성을 좁히거나 마이그레이션 테이블 collation을 다룰 때
type: convention
---

# MikroORM 엔티티·마이그레이션 함정

## nullable 타입 좁히기

- `defineEntity`의 `.nullable()` 속성은 `T | null | undefined`로 추론되어 `!== null` 비교로는 좁혀지지 않으므로 `typeof`·`instanceof`로 좁힘

## 마이그레이션

- 마이그레이션 테이블 DDL은 `default character set utf8mb4`만 붙고 collation을 지정하지 않아 서버 설정(`utf8mb4_unicode_ci`)과 달리 테이블은 MySQL 8 기본값 `utf8mb4_0900_ai_ci`로 생성됨

## 심볼 collation 의존

- 테이블 collation을 바꾸면 종목 심볼(stock.symbol) 대소문자 처리를 코드로 옮겨야 함 — unique 인덱스와 조회의 대소문자 무시가 코드가 아니라 collation(`utf8mb4_0900_ai_ci`)에 의존함

## datetime 소수 초

- 소수 초 없는 datetime 컬럼(stock.created_at)은 저장 직후 반환한 엔티티 값은 밀리초를 담고 이후 조회 값은 `.000`으로 잘려, 등록 응답과 목록 조회의 시각이 밀리초 단위로 다름

## 공유 개발 DB 마이그레이션 오염

- 모든 worktree가 공유하는 개발 DB(pigeon_trade)의 candle 테이블에는 엔티티에 없는 인덱스(candle_stock_id_index)가 있음
- 마이그레이션 스냅샷 diff에서 candle 변경을 걸러냄 — 마이그레이션 생성(migration:create)이 이 인덱스를 스냅샷에 싣거나 검사(migration:check)가 drop 마이그레이션을 제안함

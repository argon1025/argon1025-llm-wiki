---
description: MikroORM 엔티티 nullable 속성을 좁히거나 마이그레이션 테이블 collation을 다룰 때
type: convention
---

# MikroORM 엔티티·마이그레이션 함정

## nullable 타입 좁히기

- `defineEntity`의 `.nullable()` 속성은 `T | null | undefined`로 추론되어 `!== null` 비교로는 좁혀지지 않으므로 `typeof`·`instanceof`로 좁힘

## 마이그레이션

- 마이그레이션 테이블 DDL은 `default character set utf8mb4`만 붙고 collation을 지정하지 않아 서버 설정(`utf8mb4_unicode_ci`)과 달리 테이블은 MySQL 8 기본값 `utf8mb4_0900_ai_ci`로 생성됨

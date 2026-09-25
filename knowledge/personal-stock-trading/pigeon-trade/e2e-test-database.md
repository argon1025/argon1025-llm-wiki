---
description: 공유 테스트 DB에 FK로 참조된 잔존 테이블 때문에 e2e 스키마 초기화가 실패할 때
type: convention
---

# e2e 공유 테스트 DB 잔존 테이블

## 규칙

- e2e의 스키마 초기화(orm.schema.refresh())는 현재 엔티티에 없는 테이블을 지우지 않음
- 다른 브랜치·worktree가 공유 테스트 DB pigeon_trade_test에 남긴 stock 참조 FK 테이블이 있으면 `Cannot drop table 'stock' referenced by a foreign key constraint`로 e2e 전 스위트가 실패함
- 이때는 잔존 테이블을 테스트 DB에서 직접 drop함

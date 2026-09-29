---
description: vitest 테스트나 실DB e2e spec을 작성·수정·실행할 때
---

# 테스트 실행 제약

## 규칙

- Nest 앱을 띄운 테스트로 HTTP 부품을 검증하지 않고 부품별 단위 테스트로 검증함 — vitest는 esbuild 변환이라 `emitDecoratorMetadata`를 내보내지 않아 생성자 주입과 `ValidationPipe`의 DTO 판별이 동작하지 않음

## 함정

- 다른 브랜치·worktree가 테스트 DB에 stock 참조 FK 테이블을 남기면 e2e 전 스위트가 `Cannot drop table 'stock'`로 실패해 그 테이블을 직접 drop함 — `orm.schema.refresh()`는 엔티티에 없는 테이블을 지우지 않음
- 여러 worktree가 e2e를 동시에 돌리면 서로의 `orm.schema.refresh()`·`clear()`가 끼어들어 무작위로 실패함 — 모든 worktree가 테스트 DB(`pigeon_trade_test`)를 공유함

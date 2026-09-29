---
description: 마이그레이션을 생성하거나 스냅샷 diff를 커밋할 때
---

# MikroORM 마이그레이션

## 규칙

- 머지 전 브랜치 안에서 스키마 설계가 뒤집히면 마이그레이션 이력으로 남기지 않고 로컬 DB를 `migration:down`으로 되돌린 뒤 최종 스키마만 담은 마이그레이션 하나로 다시 생성함
- 마이그레이션을 커밋하면 스냅샷도 함께 커밋함 — 다음 `migration:create`가 스냅샷을 diff 기준으로 씀

## 함정

- `migration:create`가 스냅샷에 싣거나 `migration:check`가 drop 마이그레이션을 제안하는 candle 테이블의 `candle_stock_id_index` 변경은 걸러냄 — 엔티티(`index(false)`)에 없는 인덱스가 공유 개발 DB에만 남아 있음
- `npm run migration:create`가 스냅샷의 `stock.enabled` precision·scale을 3·0과 null 사이로 바꿔 쓰면 되돌리지 않고 그대로 커밋함 — 실행마다 번갈아 바뀌는 값이며 마이그레이션 DDL에는 반영되지 않음

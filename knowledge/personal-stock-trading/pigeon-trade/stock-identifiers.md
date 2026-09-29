---
description: 종목 ISIN·심볼 식별이나 토스 심볼 매핑을 다룰 때
---

# 종목 식별자

## 규칙

- `stock`을 참조하는 하위 테이블(캔들 등)은 대리키 `id`를 FK로 씀 — 티커·ISIN(`isinCode`)이 바뀌어도 참조가 흔들리지 않게 함

## 함정

- ISIN 국가 접두어로 토스 상장 시장(`market`)을 추정할 수 없음 — 토스가 외국 종목(`FOREIGN_STOCK`·`DEPOSITARY_RECEIPT`)도 국내·미국 시장에 상장시킴
- `toss_invest_symbol`에 행이 없는 상장 예정·상장폐지 종목의 ISIN은 404 `BrokerageException`이 남 — 토스 `/api/v1/stocks/all`을 상장 상태 미지정으로 조회해 `ACTIVE`만 받음
- 매핑이 24시간을 넘긴 상장폐지 종목은 요청마다 저장된 시장을 `/api/v1/stocks/all`로 재조회함(알려진 한계) — 재조회 결과에 없어 확인 시각(`updated_at`)이 갱신되지 않음
- `stock.symbol` 조회는 대소문자를 무시해 `aapl`과 `AAPL`을 같은 종목으로 봄 — 코드가 아니라 테이블 collation에 의존함, 마이그레이션이 collation을 지정하지 않아 MySQL 8 기본값(`utf8mb4_0900_ai_ci`)이 적용됨
- `stock` 테이블의 collation을 바꾸면 심볼 대소문자 처리를 코드로 옮겨야 함

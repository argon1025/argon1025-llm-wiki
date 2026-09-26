---
description: 목록 조회 쿼리 DTO를 설계할 때
type: convention
---

# 조회 요청 DTO 규칙

## 규칙

- 캔들·지표 등 목록 조회 구간은 from·to 범위 대신 before(exclusive)+limit 커서 방식을 씀 — 1m 봉이 종목당 수십만 행이라 범위 조회 거부 필요, 차트 좌측 스크롤 추가 로드에 적합
- 숫자 쿼리 파라미터는 DTO에 @Type(() => Number)를 붙여야 숫자로 변환됨 — 전역 ValidationPipe가 transform: true이나 enableImplicitConversion이 없음

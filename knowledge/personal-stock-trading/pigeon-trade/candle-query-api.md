---
description: 캔들 조회 API의 페이지네이션 방식을 정할 때
type: convention
---

# 캔들 조회 API

## 페이지네이션

- 캔들 조회는 from·to 범위 대신 before(exclusive)+limit 커서 방식임 — 1m 봉이 종목당 수십만 행이라 범위 조회는 긴 구간을 거부해야 하고, 차트 좌측 스크롤 추가 로드에 커서 방식이 맞음

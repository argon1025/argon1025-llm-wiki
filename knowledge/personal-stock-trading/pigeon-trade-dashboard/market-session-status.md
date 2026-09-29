---
description: 장 상태 배지나 다음 세션 전환 시각을 표시할 때
---

# 장 상태 표시

## 함정

- 세션 전환 직후 최대 30초(PRICE_REFETCH_INTERVAL) 동안 이전 장 상태가 보이는 것은 의도된 동작 — 응답의 현재 세션(currentSession)과 조회 시각(dataUpdatedAt)으로 계산해 렌더 중 Date.now()를 쓰지 않는 대가임

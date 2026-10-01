---
description: 종목 차트의 상승하락 색·지표 조회·pane 높이를 고칠 때
---

# 종목 차트

## 규칙

- 모든 종목(미국 종목 포함)은 국내 색 관례대로 상승 빨강(UP_COLOR)·하락 파랑(DOWN_COLOR)으로 표시함(의도된 동작)
- 현재가 변동 강조(animate-price-rise·animate-price-fall·PriceDelta)의 상승·하락 색은 차트의 UP_COLOR(#e5383b)·DOWN_COLOR(#1971c2)와 같아야 함
- 차트 지표 조회(stockQueries.indicators)는 토글과 무관하게 include에 전 지표(ALL_INDICATORS)를 보내고 토글은 그릴 시리즈만 정함 — 토글 변경이 재조회를 일으키지 않음

## 함정

- lightweight-charts v5 autoSize 차트의 보조 pane 높이는 setHeight() 대신 setStretchFactor() 비율로 나눔(StockChart) — 생성 직후에는 크기가 확정되지 않아 setHeight()로 정하면 첫 보조 pane이 눌림
- 봉 단위·지표 토글을 바꾸면 차트를 새로 만들어 확대·스크롤 위치가 초기화됨(의도된 동작) — 구현을 단순화한 것이며 위치 유지 요구가 생기면 시리즈 단위 추가·제거로 바꿈

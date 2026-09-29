---
description: 지표 조회 API를 호출하거나 지표 null·계산 규칙을 다룰 때
---

# 지표 조회 API

## 적용 대상

- 지표를 계산해 제공하는 pigeon-trade, 조회해 화면에 그리는 pigeon-trade-dashboard

## 규칙

- 지표 조회 API(GET /api/stocks/:symbol/indicators)는 interval과 include(ema20·ema50·rsi14·atr14 중 1개 이상)가 필수임
- include 생략·빈 목록·허용 밖 값이면 400 INVALID_REQUEST, 없는 심볼이면 404 STOCK_NOT_FOUND
- 요청은 before·limit(1~1000, 기본 200)을 받음
- 응답 항목마다 OHLCV·지표·indicatorVersion을 함께 담아 캔들·거래량·지표를 한 번의 호출로 그릴 수 있음
- 대상 봉까지 저장된 봉 수가 지표별 최소 봉 수(`## 코드값` 표)에 못 미치면 그 지표는 기간 미달로 null임
- 지표 값은 소수 8자리로 반올림함
- include에 없는 지표는 필드를 생략하지 않고 null로 반환함 — 기간 미달 null과 응답만으로 구분할 수 없어 소비자가 자신이 보낸 include로 구분함
- 지표는 저장하지 않고 요청 시 계산함
- 지표는 봉마다 대상 봉 포함 직전 200개 봉의 고정 윈도로 계산해 조회 범위와 무관하게 같은 봉·같은 지표 버전이면 값이 같음 — EMA·Wilder 평활은 계산 시작점에 따라 값이 달라져 조회 범위를 시작점으로 쓰면 같은 봉도 조회마다 값이 달라짐
- 지표 계산 규칙 버전(INDICATOR_VERSION, 응답 indicatorVersion)은 v2이며 지표·윈도·봉 집계 규칙이 바뀌면 올림
- INDICATOR_VERSION을 올리면 과거 규칙의 값은 다시 계산할 수 없음 — 재판정 근거는 판단 기록에 복사해 둔 입력 JSON이 담당함

## 코드값

원본: `IndicatorName` 전 4종

| 코드 | 뜻 | 최소 봉 수 |
|---|---|---|
| `ema20` | 종가 EMA20 | 20 |
| `ema50` | 종가 EMA50 | 50 |
| `rsi14` | 종가 RSI14, Wilder 평활이며 평균 상승·하락이 모두 0이면 50 | 15 |
| `atr14` | ATR14, True Range는 직전 봉 종가가 있는 두 번째 봉부터 계산 | 15 |

## 함정

- 1h 지표의 200봉 윈도는 미국 약 8일·국내 약 17일 분량이며 전략 프롬프트가 이 기간을 감안함 — 1h 봉이 시간외를 포함해 집계됨

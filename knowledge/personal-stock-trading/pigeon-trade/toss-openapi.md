---
description: 토스 Open API 명세를 확인하거나 시세 응답·한도를 해석할 때
---

# 토스 Open API 명세

## 규칙

- 토스 1d 캔들의 nextBefore를 다음 요청 before에 그대로 넘기면 봉 중복 없이 이어짐 — nextBefore는 페이지 마지막 봉의 직전 거래일 현지 자정(주말·휴장일 건너뜀)이고 before는 inclusive임
- 1m을 과거에서 앞으로 쌓으려면 before를 워터마크 + 200분(1m 200봉)으로 요청함 — 토스 캔들 조회(GET /api/v1/candles)는 before·count(최대 200)로 과거 방향 페이지만 받고 시작 시각을 주는 순방향 조회가 없음
- 토스 캔들 조회(GET /api/v1/candles)의 1m은 before 페이지네이션으로 최소 2026-07-31까지 과거를 돌려줌
- 토스 캔들 조회로 받은 과거 1m을 저장 30m 집계와 같은 UTC 버킷으로 30m 집계하면 종가가 저장된 30m 봉과 일치함
- 토스 종목정보 조회(GET /api/v1/stocks) 결과에 요청 심볼이 없으면 없는 종목(STOCK_NOT_FOUND)으로 판정함 — 토스가 존재하지 않는 심볼을 오류 없이 result에서 빼고 응답함
- 토스 시세 API는 존재하지 않는 종목이면 404와 error.code stock-not-found를 반환함
- 토스 캔들 조회(GET /api/v1/candles, MARKET_DATA_CHART)의 한도는 초당 20회(X-RateLimit-Limit 20, Reset 1)이며 성공 응답에도 X-RateLimit-* 헤더가 붙음
- 토스 캔들 조회가 한도를 넘으면 429·Retry-After 1·error.code rate-limit-exceeded를 반환함
- 토스 현재가·호가·체결·상하한가 조회는 캔들(MARKET_DATA_CHART)과 별도인 MARKET_DATA 그룹이며 한도는 초당 15회로 캔들과 따로 계산됨
- 토스 1d 봉은 미국 종목이면 정규장 시가·고가·저가·종가, 국내 종목이면 NXT 프리·애프터마켓을 포함한 값임

## 함정

- 토스 시세 API가 반환하는 시각은 미국 종목도 +09:00 오프셋이며 거래소 시간대가 아님 — 거래소 현지 시각은 종목 시장에 맞춰 setZone으로 따로 변환함
- 토스 미국 장 운영 캘린더(GET /api/v1/market-calendar/US)의 today를 미국 현지 당일로 가정하면 틀림 — date를 생략하면 KST 날짜를 당일로 잡아 KST 자정 이후 진행 중인 미국 세션은 previousBusinessDay에 속함
- 토스 미국 1m을 모아도 토스 1d와 같아지지 않음 — 미국 1m은 전 거래소 통합 체결 기준이 아님

## 외부 참조

| 리소스 | 접근 방법 | 참조 시점 |
|---|---|---|
| 토스 Open API 정본 명세 | https://openapi.tossinvest.com/openapi-docs/latest/openapi.json — 개발자센터 llms.txt가 정본으로 지정함 | 토스 API 요청·응답 형식을 확인할 때 |

- 토스 Open API 명세에는 레이트 리밋 수치가 없고 429 응답 헤더(X-RateLimit-Limit·X-RateLimit-Remaining·X-RateLimit-Reset·Retry-After)와 API별 Rate Limits Group만 정의됨

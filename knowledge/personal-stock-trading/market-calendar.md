---
description: 장 운영 일정이나 현재 세션(currentSession)을 조회·소비할 때
---

# 장 운영 일정

## 적용 대상

- 일정을 동기화·제공하는 pigeon-trade, 일정을 소비하는 pigeon-trade-dashboard

## 규칙

- 장 운영 일정은 REST GET /api/market-calendars/{country}와 MCP 도구 get_market_calendar({ country })로 조회함
- country가 KR·US 외 값이면 REST는 400(INVALID_REQUEST), MCP는 입력 검증 오류를 반환함
- 국가 구분 필드·인자 이름은 `market`이 아니라 `country`(Country) — 종목 market(KOSPI 등 거래소 단위)과 같은 이름에 다른 값을 담는 충돌을 피함
- country KR은 market이 KOSPI·KOSDAQ·KR_ETC인 종목, US는 NYSE·NASDAQ·AMEX·US_ETC인 종목의 일정임 — 종목 조회 결과의 market으로 country를 골라야 함
- 응답의 days는 가장 최근 동기화한 전 영업일·당일·익 영업일과 그 사이 휴장일을 날짜 오름차순으로 담음
- 휴장일은 sessions가 빈 배열이고 open이 false임
- 응답의 세션은 DAY·PRE·REGULAR·AFTER 4종의 시작·종료 시각(UTC ISO)만 담고 토스 캘린더의 단일가 구간 시각(singlePriceAuctionStartTime·singlePriceAuctionEndTime)은 버림
- 국내(KR)는 KRX·NXT 통합 기준 PRE·REGULAR·AFTER 3세션이며 장전·장후 시간외종가는 제외함
- 현재 세션(currentSession)은 서버가 요청 시각 기준으로 계산해 REST·MCP 응답에 담음 — 소비자가 KST·현지·UTC 시각을 직접 비교하면 오판 위험이 있어 소비자 계산안은 기각함
- 휴장 여부는 거래소 현지 오늘 날짜의 days[].open이 false인지로 가림 — currentSession CLOSED는 휴장일과 영업일의 세션 사이를 구분하지 않음

## 코드값

원본: `Country` 전 2종

| 코드 | 뜻 |
|---|---|
| `KR` | 국내 시장의 장 운영 일정, 영업일 date는 KST 기준 |
| `US` | 미국 시장의 장 운영 일정, 영업일 date는 미국 현지(동부) 날짜라 거래일 D의 세션은 KST D 09:00 데이마켓 시작부터 KST D+1 애프터마켓 종료까지 걸침 |

원본: `MarketSessionStatus` 전 6종

| 코드 | 뜻 |
|---|---|
| `DAY` | 진행 중인 세션이 미국 데이마켓, 미국 시장에만 있음 |
| `PRE` | 진행 중인 세션이 프리마켓 |
| `REGULAR` | 진행 중인 세션이 정규장 |
| `AFTER` | 진행 중인 세션이 애프터마켓 |
| `CLOSED` | 진행 중인 세션이 없지만 저장된 일정에 이후 세션이 남아 있음, 휴장일과 세션 사이를 구분하지 않음 |
| `UNKNOWN` | 저장된 일정이 현재 시각을 덮지 못함(일정이 하나도 없을 때 포함), null 대신 이 값을 반환함 |

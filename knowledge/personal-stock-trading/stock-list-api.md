---
description: 종목을 등록하거나 종목 목록·현재가 응답을 다룰 때
---

# 종목 등록과 목록 API

## 적용 대상

- 종목 API를 제공하는 pigeon-trade, 목록 응답을 소비하는 pigeon-trade-dashboard

## 규칙

- 종목 테이블(stock)은 시세 동기화 대상 종목 풀이며 enabled가 false인 종목은 시세 수집에서 제외됨
- 종목 등록 요청(POST /api/stocks)은 enabled를 필수로 받음
- 종목 심볼(stock.symbol)은 API 사용자가 종목을 지정하는 서비스 고유 심볼임
- 종목 등록 API는 임시로 토스 심볼을 받아 그대로 심볼로 저장하고 ISIN을 찾아 함께 저장함
- 종목 등록은 이미 등록된 심볼·같은 ISIN·동시 요청 unique 위반이면 409(STOCK_ALREADY_REGISTERED)로 거부함
- 종목 등록은 토스에 없는 종목이면 404(STOCK_NOT_FOUND), 상장 상태(ACTIVE)가 아니면 400(STOCK_NOT_ACTIVE)으로 거부함
- 종목 등록은 시장(market)·통화(currency)를 토스 종목 정보로 채우고 시간대(stock.timezone)는 코드값 표의 시장별 시간대로 채움
- 종목 API는 POST /api/stocks(등록)와 GET /api/stocks(목록)임
- 종목 응답의 등록 시각(createdAt)은 UTC ISO 문자열임
- 종목 응답의 id는 JSON number이며 pigeon-trade-dashboard의 Stock.id 타입도 number임 — pigeon-trade가 bigint 대리키를 number로 직렬화함
- GET /api/stocks는 페이지네이션·검색·정렬 파라미터 없이 전체 종목을 id 오름차순(등록순)으로 반환함 — 검색·정렬·분할은 클라이언트에서만 가능함
- 종목 단건 조회 API는 없음 — 한 종목이 필요하면 GET /api/stocks 목록에서 symbol로 찾음
- GET /api/stocks와 MCP list_stocks 응답 항목은 현재가(price, 최신 1m 봉 종가 decimal 문자열)와 기준 봉 시각(priceTimestamp, UTC ISO 봉 종료 시각)을 담으며 1m 봉이 없는 종목은 둘 다 null임
- 종목 등록 응답(POST /api/stocks)에는 price·priceTimestamp가 없음
- 종목 현재가(stock.price)는 1m 저장 이벤트(candle.1m.saved)가 실은 최신 1m 봉 종가·봉 시각으로만 갱신됨
- pigeon-trade의 종목 목록 응답에는 전일 종가·등락률 필드가 없어 pigeon-trade-dashboard는 일간 등락률을 계산할 수 없음 — 일간 등락률 표시에는 pigeon-trade의 필드 추가가 먼저 필요함

## 코드값

원본: `Market` 전 7종

| 코드 | 뜻 | 시간대 |
|---|---|---|
| `KOSPI` | 국내 시장 | `Asia/Seoul` |
| `KOSDAQ` | 국내 시장 | `Asia/Seoul` |
| `KR_ETC` | 국내 기타 시장 | `Asia/Seoul` |
| `NYSE` | 미국 시장 | `America/New_York` |
| `NASDAQ` | 미국 시장 | `America/New_York` |
| `AMEX` | 미국 시장 | `America/New_York` |
| `US_ETC` | 미국 기타 시장 | `America/New_York` |

- 위 코드는 stock.market에 저장됨

## 함정

- 이미 등록된 상장폐지 종목의 티커를 다른 종목이 재사용하면 그 종목 등록이 409(STOCK_ALREADY_REGISTERED)로 막힘 — 심볼이 서비스 고유값이라 등록된 심볼과 충돌함
- 종목 등록 응답의 createdAt은 밀리초를 담지만 이후 조회는 초 단위로 반올림된 .000Z 값으로 나옴
- 이벤트가 나가지 않는 장외·휴장·enabled=false 종목과 이미 저장된 최신 봉의 가격 보정 upsert에서는 현재가가 바뀌지 않아 candle 최신 종가와 다를 수 있음 — 다음 새 봉 저장 시 해소됨
- 이벤트 처리 순서가 뒤바뀌면 현재가(price·priceTimestamp)가 더 이른 봉 값으로 덮어써질 수 있음(의도된 동작) — 현재가는 참고 지표라 순서 역전을 막지 않음
- 종목 현재가는 장중 1~2분 늦고 장외에는 시간외 마지막 봉 값(국내 종목은 20:00 KST 봉)이라 정규장 종가와 다를 수 있음 — 완료된 1m 봉만 반영함
- 지갑 포지션 평가·주문 체결가는 종목 목록 현재가(stock.price)와 1분 이내 시차가 날 수 있음 — 둘은 조회 시점에 candle의 최신 1m 종가를 직접 읽음

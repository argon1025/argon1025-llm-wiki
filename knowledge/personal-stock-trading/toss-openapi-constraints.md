---
description: 토스증권 Open API로 시세를 받거나 리플레이 구간을 정할 때
type: external
---

# 토스증권 Open API 제약

## 적용 대상

- MVP는 토스를 시세 공급원으로만 사용하며 아래 제약만 영향을 줌

## 제약

| 항목 | 제약 | 대응 |
| --- | --- | --- |
| 토큰 | 클라이언트당 유효 토큰 1개 | 백엔드가 독점 |
| 허용 IP | 호출 IP 변경 시 차단 | 홈서버 운영이라 고정 공인 IP 또는 중계 필요, 확보 방법은 아직 정해지지 않음 |
| 과거 데이터 | 국내 2022-11-23, 미국 2021-11-30 이후 | 리플레이 구간은 이 범위 안 |
| 미국 시세 | 일부 거래소 기준, 1분봉 이상값 가능 | 지표 계산 전 이상값 처리 |
| 데이터 이용 | 본인 매매 목적만 | 대시보드 본인 전용 |
| 레이트 리밋 | 명세에 수치 없음, 429 응답 헤더(`X-RateLimit-Limit`·`X-RateLimit-Remaining`·`X-RateLimit-Reset`·`Retry-After`)만 정의, Rate Limits Group은 캔들 `MARKET_DATA_CHART`·종목 정보 조회 `STOCK`·상장 종목 목록 조회 `STOCK_ALL` | 실제 한도는 실계정 응답 헤더로 측정 |

## API 형식

- 정본 명세는 `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json` — 개발자센터 `llms.txt`가 이 경로를 정본으로 지정함
- 4xx·5xx 오류 본문은 `{ error: { requestId, code, message, data? } }` 형식이나 토큰 발급(`POST /oauth2/token`) 실패만 `{ error: 'invalid_client', error_description }` 형식
- 통화 코드(`Currency`)는 `KRW`·`USD`이나 명세가 알 수 없는 enum 값 허용을 요구함
- 여러 클라이언트에서 토큰을 발급하면 서로의 토큰을 끊음 — 토큰 발급(`POST /oauth2/token`)은 재발급 즉시 이전 토큰을 무효화하고 refresh token이 없음
- 시세 API 토큰 오류는 401과 `error.code`(`invalid-token`·`expired-token`·`token-revoked`·`login-user-not-found`)로 오며 `token-revoked`는 다른 곳의 재발급으로 무효화된 토큰을 뜻함
- 존재하지 않는 종목 조회 시 404와 `error.code` `stock-not-found`를 반환함

## 종목 조회

- 조회 API는 종목을 증권사 심볼(`symbol`·`symbols`)로만 받고 ISIN(`isinCode`)은 종목 정보 조회(`GET /api/v1/stocks`)·상장 종목 목록 조회(`GET /api/v1/stocks/all`) 응답에만 있음
- 종목 정보 조회(`GET /api/v1/stocks`, `getStocks`)는 최대 200건 다건 조회이며 명세에 404가 없고 존재하지 않는 심볼(예 `ZZZZ999`)을 오류 없이 결과에서 빼고 응답함 — 결과 누락을 없는 종목으로 판정해야 함(실호출 확인)
- 상장 종목 목록 조회(`GET /api/v1/stocks/all`, `listStocks`)는 `market` 필수이고 `status` 미지정 시 `ACTIVE`만 반환함
- 상장 종목 목록 조회는 하루 1회 조회 후 캐싱이 권장됨 — 페이지네이션 없이 마켓당 최대 수천 건을 주고 일 배치로 갱신됨
- 외국 종목(`securityType` `FOREIGN_STOCK`·`DEPOSITARY_RECEIPT`)도 국내·미국 시장에 상장되므로 ISIN 국가 접두어로 토스 시장(`market`)을 추정할 수 없음
- 토스 심볼은 KRX 6자리 종목코드(숫자 또는 영문·숫자 조합, 예 `0101N0`)와 미국 영문 티커이고 허용 패턴(`^[A-Za-z0-9.\-]+`)만으로 국내·미국을 가를 수 없으며 응답의 `market`으로 판정해야 함

## 캔들 데이터

- 캔들 수집·리플레이에 정규장(15:30) 이후 시간외 봉이 섞임 — 1분봉(1m)은 20:00 KST 봉까지 제공됨
- 일봉(1d)은 1분봉(1m)을 모아 만들 수 없어 별도로 수집해야 함 — 1m에 20:00 KST까지 시간외 봉이 섞이고 시가·종가는 단일가로 정해지며 미국 1m은 일부 거래소 기준임
- 일봉(1d) 다음 페이지 커서(`nextBefore`)는 페이지 마지막 봉의 직전 거래일 현지 자정(휴장일 건너뜀)이고 `before`는 inclusive라 받은 값을 그대로 넘기면 봉 중복 없이 이어짐
- 거래소 현지 시각은 응답 오프셋이 아닌 종목 시장 시간대로 따로 변환해야 함 — 미국 종목(AAPL 등)도 시각을 +09:00 오프셋으로 반환함

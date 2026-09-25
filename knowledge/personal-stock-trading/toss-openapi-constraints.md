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

## API 형식

- 정본 명세는 `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json` — 개발자센터 `llms.txt`가 이 경로를 정본으로 지정함
- 4xx·5xx 오류 본문은 `{ error: { requestId, code, message, data? } }` 형식이나 토큰 발급(`POST /oauth2/token`) 실패만 `{ error: 'invalid_client', error_description }` 형식
- 통화 코드(`Currency`)는 `KRW`·`USD`이나 명세가 알 수 없는 enum 값 허용을 요구함
- 여러 클라이언트에서 토큰을 발급하면 서로의 토큰을 끊음 — 토큰 발급(`POST /oauth2/token`)은 재발급 즉시 이전 토큰을 무효화하고 refresh token이 없음
- 시세 API 토큰 오류는 401과 `error.code`(`invalid-token`·`expired-token`·`token-revoked`·`login-user-not-found`)로 오며 `token-revoked`는 다른 곳의 재발급으로 무효화된 토큰을 뜻함
- 존재하지 않는 종목 조회 시 404와 `error.code` `stock-not-found`를 반환함
- 종목 정보 조회(`GET /api/v1/stocks`)만 예외로 존재하지 않는 심볼을 404 없이 `result`에서 빼고 응답함 — 명세 응답 코드에도 404가 없음(실호출 확인 2026-09-25)

## 종목 식별

- 모든 조회는 증권사 심볼(`symbol`·`symbols`)로만 받고 국제증권식별번호(isinCode)는 종목 정보 조회(`GET /api/v1/stocks`)·상장 종목 목록(`GET /api/v1/stocks/all`) 응답에만 있음

## 레이트 리밋

- 레이트 리밋 수치는 명세에 없고 429 응답 헤더(X-RateLimit-Limit·X-RateLimit-Remaining·X-RateLimit-Reset·Retry-After)로 정의되며 이 헤더는 성공 응답에도 붙음 (실호출 확인 2026-09-25)
- 레이트 리밋 초과 시 429·Retry-After: 1·`error.code` `rate-limit-exceeded`를 반환함 (실호출 확인 2026-09-25)
- 레이트 리밋 그룹(Rate Limits Group)은 캔들 조회(`GET /api/v1/candles`) MARKET_DATA_CHART·현재가·호가·체결·상하한가 조회 MARKET_DATA·종목 정보 STOCK·상장 종목 목록 STOCK_ALL로 나뉨
- 레이트 리밋 한도는 캔들 조회(MARKET_DATA_CHART) 초당 20회(X-RateLimit-Reset: 1)·현재가·호가·체결·상하한가 조회(MARKET_DATA) 초당 15회이고 두 그룹은 따로 계산됨 (실호출 확인 2026-09-25)

## 캔들 데이터

- 캔들 수집·리플레이에 정규장(15:30) 이후 시간외 봉이 섞임 — 1분봉(1m)은 20:00 KST 봉까지 제공됨
- 1분봉(1m)을 모아도 일봉(1d)이 되지 않음 — 1m에 20:00 KST까지 시간외 봉이 섞이고 시가·종가는 단일가로 정해지며 미국 1m은 일부 거래소 기준임
- 캔들 조회 경계 시각(`before`)은 밀리초(19:58:59.999+09:00)와 UTC Z 표기(10:58:59.999Z)를 모두 받아 1m·1d 모두 같은 결과를 줌 (실호출 확인 2026-09-26)
- 국내 종목 1분봉(1m)은 거래일당 08:01~20:00 KST 720봉임 (실호출 확인 2026-09-25)
- 1분봉(1m) 다음 페이지 커서(`nextBefore`)는 페이지 마지막 봉 시각의 1분 전(거래일 경계에서는 직전 거래일 20:00 KST)이라 그대로 넘기면 페이지 간 중복 없음 (실호출 확인 2026-09-25)
- 일봉(1d) 다음 페이지 커서(`nextBefore`)는 페이지 마지막 봉의 직전 거래일 현지 자정(휴장일 건너뜀)이고 `before`는 inclusive라 받은 값을 그대로 넘기면 봉 중복 없이 이어짐
- 거래소 현지 시각은 응답 오프셋이 아닌 종목 시장 시간대로 따로 변환해야 함 — 미국 종목(AAPL 등)도 시각을 +09:00 오프셋으로 반환함

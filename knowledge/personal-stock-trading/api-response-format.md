---
description: HTTP API 경로·본문 한도·응답 형식이나 응답 코드를 다룰 때
---

# API 응답 형식

## 적용 대상

- pigeon-trade의 HTTP API

## 규칙

- HTTP API는 경로 prefix `/api`를 쓰고 버전은 붙이지 않음
- 모든 응답은 `{ success, responseCode, message, data }` 형식이며, 성공이면 responseCode `OK`·message `성공`으로 고정됨
- 실패 응답은 data `null`이며 HTTP status는 응답 코드마다 정해짐
- 요청 검증은 정의되지 않은 필드를 거부(forbidNonWhitelisted)하고, 실패 사유를 `{필드 경로}: {사유} / {필드 경로}: {사유}` 형식의 message로 합쳐 `INVALID_REQUEST`(400)로 응답함
- 가격·거래량(decimal 24,8) 같은 decimal 응답 값은 JSON number가 아니라 `Decimal#toFixed()` 문자열로 내보내고, 새 decimal 응답도 같은 방식을 따름 — JSON number로는 정밀도가 손실됨
- JSON 요청 본문 한도는 10mb — Express 기본 100kb에서는 주입 컨텍스트 전문을 받는 판단 기록 POST가 실패함

## 코드값

원본: `ResponseCode` — 일부

| 코드 | 뜻 | HTTP status |
|---|---|---|
| `OK` | 성공 응답(message `성공`) | 200(POST는 201) |
| `INVALID_REQUEST` | 요청 값 검증 실패 또는 일반 400 응답, message에 실패 필드별 사유가 담길 수 있음 | 400 |
| `NOT_FOUND` | 코드가 지정되지 않은 404 응답(요청한 리소스를 찾을 수 없음) | 404 |
| `STOCK_NOT_FOUND` | 존재하지 않는 종목 — 종목 등록은 토스에서 조회되지 않는 심볼, 그 밖의 API는 미등록 심볼 | 404 |
| `STOCK_NOT_ACTIVE` | 상장 상태(ACTIVE)가 아닌 종목이라 등록할 수 없음 | 400 |
| `STOCK_ALREADY_REGISTERED` | 이미 등록된 심볼 또는 ISIN이라 등록할 수 없음 | 409 |
| `BROKERAGE_BAD_REQUEST` | 증권사가 요청을 400으로 거부함 | 400 |
| `BROKERAGE_NOT_FOUND` | 증권사에서 대상을 찾을 수 없음 | 404 |
| `BROKERAGE_UNAVAILABLE` | 증권사 인증·한도·서버 오류·타임아웃 등 연동 실패 | 502 |
| `INTERNAL_SERVER_ERROR` | 분류되지 않은 서버 오류, message는 내부 정보 없이 고정 문구 | 5xx |
| `WALLET_NOT_FOUND` | 존재하지 않는 지갑 | 404 |
| `WALLET_ALREADY_EXISTS` | 지갑 이름 중복 | 409 |
| `INSUFFICIENT_CASH` | 종목 통화 잔고 행이 없거나 체결 금액+수수료가 현금 초과 | 400 |
| `INSUFFICIENT_QUANTITY` | 매도 수량이 보유 수량 초과 | 400 |
| `ORDER_NOT_FOUND` | 판단 기록의 orderId가 해당 지갑에 없는 주문 | 404 |
| `DECISION_NOT_FOUND` | 지갑에 없는 판단 식별자를 상세 조회함 | 404 |

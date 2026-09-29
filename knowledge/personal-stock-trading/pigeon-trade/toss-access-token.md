---
description: 토스 액세스 토큰을 발급·재발급하거나 보관 방식을 다룰 때
---

# 토스 액세스 토큰

## 규칙

- 토큰 발급(`TossInvestAccessTokenService`)은 프로세스 안에서 발급 Promise를 공유하고 프로세스 간에는 `client_id` 행 잠금으로 직렬화함 — 토스는 클라이언트당 유효 토큰 1개만 허용해 재발급 시 이전 토큰을 즉시 무효화함

## 코드값

원본: 토스 시세 API 401 응답 `error.code` — 일부

| 코드 | 뜻 |
|---|---|
| `invalid-token` | 뜻 미확인 |
| `expired-token` | 뜻 미확인 |
| `token-revoked` | 다른 곳의 재발급으로 무효화된 토큰 |
| `login-user-not-found` | 뜻 미확인 |

## 함정

- 토스 토큰 발급(`POST /oauth2/token`) 실패 응답은 `{ error: 'invalid_client', error_description }`라 `error.code`가 없는 것이 정상 — 토큰 발급만 공통 envelope 없는 OAuth2 표준 형식임

## 결정

- 토큰 사전 갱신 스케줄러를 두지 않고 시세 호출 시점에 만료 5분 전(`TOKEN_REFRESH_MARGIN`)이면 재발급함 — 스케줄러 의존성과 미사용 시간대 발급을 피함
- 토스 액세스 토큰을 메모리에만 두지 않고 DB(`toss_invest_access_token`)에 보관함 — 재기동 시 재사용과 여러 프로세스의 토큰 공유가 목적임

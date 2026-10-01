---
description: 증권사 시세 구현체를 고르거나 교체하고 증권사 예외를 다룰 때
---

# 증권사 시세 전략

## 규칙

- 증권사 시세 인터페이스(`BrokerageMarketData`)는 토스 원형 타입을 재사용하지 않고 증권사 중립 타입(`Decimal`, `DateTime`)으로 반환함 — 증권사 교체 시 호출자 코드가 바뀌지 않게 함
- 증권사 시세 전략이 던지는 `BrokerageException`의 status는 증권사 응답 400·404면 같은 값, 401·403·429·5xx·타임아웃·응답 변환 실패면 502임 — 후자는 서비스 쪽 인증·외부 장애임
- 증권사 요청 한도 초과(429)는 `BrokerageException`의 status를 502로 유지한 채 `rateLimited` 속성으로 구분함 — 종목 등록 API의 응답 계약(502 `BROKERAGE_UNAVAILABLE`)을 바꾸지 않고 호출자가 증권사 중립 속성으로 판정하게 함
- 캔들 페이지 크기(count 200)는 시세 동기화 서비스가 `CANDLE_PAGE_SIZE` 상수로 관리해 매 조회에 넘기고, 증권사 계층은 최대 봉 수 같은 호출 제한을 관리하지 않음

## 함정

- 토스 전략(`toBefore`)이 캔들 조회 before를 1밀리초 앞당긴 UTC ISO로 넘기는 것은 의도된 동작 — 토스 before는 경계 시각을 포함하고 증권사 중립 before는 포함하지 않으며, 토스는 밀리초와 UTC Z 표기를 모두 받음
- 증권사 구현체를 바꾸면 `CANDLE_PAGE_SIZE`를 새 상한에 맞춰 조정해야 함 — 증권사 계층이 호출 제한을 관리하지 않음

## 결정

- 구현체 선택은 `BROKERAGE_PROVIDER`와 Nest `useFactory`로 하고 모듈 코드의 `useClass` 직접 지정안은 버림
- `BrokerageException` status를 전부 502로 통일하는 안은 버림 — 없는 종목을 호출자가 status로 구분할 수 없음

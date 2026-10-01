---
description: 도메인 모듈 폴더를 나누거나 모듈 사이 의존·공용 타입을 정할 때
---

# 도메인 모듈 경계

## 규칙

- 도메인 모듈(`src/{domain}/`)은 `controller/`·`service/`·`scheduler/`·`listener/` 계층별 폴더에 클래스를 두고 `dto/`·`interface/`·`entity/`와 모듈 파일은 도메인 루트에 두며, test도 같은 폴더 구조로 미러링함
- 지갑 도메인(`pigeon-wallet`)은 다른 도메인의 `Stock`·`Candle` 엔티티를 읽기 전용으로만 import함 — 지갑이 브로커·캔들 등 다른 도메인에 영향을 주지 않게 함
- `pigeon-wallet`이 최신 1m 종가를 `Candle` 엔티티 직접 조회로 얻는 것은 의도된 동작 — 조회성 동작이라 시세 도메인 서비스로 감싸지 않음
- 도메인 모듈은 다른 모듈이 실제 호출하는 메서드만 본 서비스로 위임하는 `{Domain}ExternalService` 하나만 export하며 이 서비스에는 로직을 두지 않음
- `AgentMcpService` 같은 소비 모듈은 다른 도메인의 본 서비스를 직접 import하지 않고 `{Domain}ExternalService`(예: `StockExternalService`)를 통해 호출함
- `src/common/type/`에는 여러 도메인이 공유하는 값 타입(`Currency`·`Market`·`Country`)만 두고, 엔티티 교차 참조·응답 DTO·서비스 결과 타입(`*Result`)은 소유 도메인에 둠
- `src/integration/**`은 common 타입을 참조하지 않고 자체 타입(`TossInvestCurrency`·`TossInvestMarket`)을 유지함 — 외부 사양 변경에 따라 바뀌는 계층임
- `src/brokerage/`는 common 타입 참조를 허용함 — integration이 아닌 서비스 쪽 중립 추상 계층임

## 함정

- unit·e2e spec은 서비스를 `new`로 직접 생성해 모듈 `providers`·`exports` 누락을 잡지 못함 — 모듈 연결을 바꾸면 앱을 로컬 기동해 MCP 도구 `list_stocks`·`get_wallet` 같은 실제 호출로 확인함

## 결정

- 시세 도메인 폴더를 기능별 폴더로 나누는 안은 기각함
- `{Domain}ExternalService` 대신 토큰·인터페이스·DI 파사드로 설계하는 안은 기각함 — 단순한 external service 파일 하나로 충분함

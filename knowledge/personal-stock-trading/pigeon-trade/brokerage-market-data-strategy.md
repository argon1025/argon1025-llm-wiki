---
description: 증권사 시세 연동을 새로 만들거나 구현체를 바꿀 때
type: convention
---

# 증권사 시세 연동 구성

## 중립 타입

- 증권사 시세 인터페이스(BrokerageMarketData)는 증권사 중립 타입(Decimal, luxon DateTime, 불투명 cursor)으로 반환함 — 토스 원형 타입 재사용안은 증권사 교체 시 호출자 코드가 바뀌어 기각
- 중립 통화 타입(BrokerageCurrency)은 'KRW' | 'USD' 닫힌 유니온으로 유지함 — PR #3 리뷰의 개방 유니온 제안은 기각

## 구현체 선택과 위치

- 증권사 구현체 선택은 BROKERAGE_PROVIDER 환경변수와 Nest useFactory로 함 — 모듈 코드에서 useClass로 직접 지정하는 안은 기각
- 새 증권사 구현체는 src/brokerage/strategy/{provider}/에 두고 클래스명은 *Strategy로 짓고 토큰 발급·보관은 구현체 내부에서 처리함
- 증권사 구현체의 MikroORM 엔티티는 서비스 옆이 아니라 src/brokerage/strategy/{provider}/entity/ 하위에 둠

## 예외 처리

- 시세 전략 실패(BrokerageException)에서 증권사 응답 400·404까지 502로 변환하는 안은 기각 — 없는 종목을 호출자가 status로 구분할 수 없음

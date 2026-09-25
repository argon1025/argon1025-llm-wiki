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
- 종목 정보 조회(getStocks)는 별도 인터페이스 대신 시세 인터페이스(BrokerageMarketData)에 추가함 — 구현체가 증권사당 하나이고 같은 토큰·예외 흐름을 써서 별도 인터페이스는 구현 하나뿐인 추상화임

## 전략 요청 콜백

- 토스 전략의 공통 요청 흐름(TossInvestMarketDataStrategy.request)에서 응답 변환 콜백(convert)은 동기 변환 전용임 — await가 필요한 부수 작업(매핑 저장 등)은 호출 콜백(call) 안에 둬야 실패가 BrokerageException으로 변환되고 401 재발급 재시도에 포함됨

## 예외 처리

- 시세 전략 실패(BrokerageException)에서 증권사 응답 400·404까지 502로 변환하는 안은 기각 — 없는 종목을 호출자가 status로 구분할 수 없음

## 종목 식별

- 시세 인터페이스(BrokerageMarketData)의 종목 입력은 국제증권식별번호(ISIN)이고 증권사 심볼 변환은 구현체 책임임 — 증권사 교체 시 호출자 코드가 바뀌지 않음
- 종목 정보 조회(getStocks)만 증권사 심볼을 받음 — ISIN을 모르는 종목 등록 시점의 편의 경로
- ISIN·증권사 심볼 매핑은 증권사 전용 테이블로 src/brokerage/strategy/{provider}/entity/에 둠 — provider 공용 매핑 테이블안은 기각
- 토스 매핑 행이 없는 ISIN은 토스 7개 시장 전체를 재조회함 — 토스는 외국 종목도 국내·미국 시장에 상장시켜 ISIN 국가 접두어로 시장을 추정할 수 없음
- 캔들 등 하위 테이블의 종목 참조는 ISIN 대신 종목 대리키(stock.id)를 씀 — ISIN도 기업 재편 시 바뀔 수 있음
- 수집 배치(Cron)는 종목의 ISIN(stock.isin_code)으로, 사용자 API는 서비스 고유 심볼을 받아 종목 테이블에서 ISIN을 찾아 시세 인터페이스를 호출함

---
description: 외부 API 연동 폴더 구조나 클라이언트·예외 모양을 설계할 때
type: convention
---

# 외부 연동 계층 구성

## 폴더 구조

- 연동 폴더는 전송 계층(`client/`)과 문서 태그별 API 클래스(`api/`)를 처음부터 분리함 — 단일 클라이언트 클래스안은 기각
- 상위 계층이 import하는 module·exception은 연동 루트에 둠 — exception을 `client/`에 두는 안은 기각
- 인터페이스는 사용하는 레이어 폴더 하위(`api/interface/`, `client/interface/`)에 둠 — 연동 루트 `interface/`는 여러 레이어가 공유하는 타입이 생길 때만 만듦
- 명세 인터페이스 파일은 외부 문서 태그 단위로 나눔 — API 1종당 1파일안은 기각
- 공식 SDK가 있어도 자체 fetch 클라이언트(client/·api/ 분리)로 구현함 — SDK의 재시도·에러 클래스 이점보다 의존성 추가와 예외 변환 래퍼 부담이 큼

## 명세 타입 명명

- 명세 타입 이름은 `TossInvest{Action}{Request|Response|Result}`이고 `{Action}`은 `operationId`의 PascalCase임
- 명세 타입 필드명은 명세 JSON 키 그대로(snake_case 포함) 두고 `required`에 없는 필드는 `?`, enum은 문자열 유니온으로 정의함

## 응답 처리

- 연동 계층은 외부 서버 응답을 envelope과 decimal 문자열까지 원형 그대로 반환하고 변환은 상위 계층 책임으로 둠 — `decimal.js`는 상위 계층용
- 외부 성공 응답은 런타임 스키마 검증 없이 인터페이스 타입을 신뢰함 — zod 검증은 의존성 추가와 인터페이스·스키마 이중 정의로 기각

## 예외 처리

- 연동 예외는 공통 부모(`IntegrationException`) 아래 연동별 단일 예외(`TossInvestException`)만 두고 실패 유형은 status 유무와 응답 본문으로 구분함 — HTTP·타임아웃·네트워크 유형별 하위 예외는 기각
- 연동 예외의 요청 헤더·바디는 `Authorization` 토큰과 `client_secret`까지 원문 그대로 담으므로 로그 외부 전송이나 운영 배포 전에 마스킹을 먼저 도입해야 함 — 로컬 프로젝트라 에러 파악 우선
- 연동 계층은 429·529 등에도 재시도 없이 예외만 발행함 — 재시도 필요 여부는 상위 계층에서 결정

## 인증

- 토스 시세 API는 액세스 토큰(`accessToken`)을 인자로 받고 연동 계층은 토큰을 보관하지 않음 — 메모리 캐시안은 만료 처리 없는 상태 저장과 `TossInvestAuthApi`·`TossInvestHttpClient` 순환 의존으로 기각
- 액세스 토큰 발급·보관·만료 갱신은 상위 계층 책임임
- 연동 자격증명(`TOSS_INVEST_CLIENT_ID`, `TOSS_INVEST_CLIENT_SECRET`)은 `ConfigService`로 조회함 — `process.env` 직접 조회와 호출자 인자 전달안은 기각

## 테스트

- 연동 테스트는 client·api 계층 단위 테스트만 두고 실제 외부 서버를 호출하는 스모크 테스트는 외부 의존이라 두지 않음

---
description: 외부 연동을 추가하거나 토스 API 호출부를 다룰 때
---

# 외부 연동 계층 구성

## 폴더와 레이어

- 연동 폴더 안은 전송 계층(client/)과 API 클래스(api/)로 나누고, 공개 계약(module·exception)만 연동 루트에 둠 — 단일 클라이언트 클래스안·client/ exception 배치안 기각
- 인터페이스는 쓰는 레이어 폴더 아래(api/interface/, client/interface/)에 두고, 연동 루트의 interface/는 여러 레이어가 공유하는 타입이 생길 때만 만듦
- 명세 인터페이스 파일은 외부 문서의 태그 단위로 한 파일씩 나눔 — API 1종당 1파일안은 기각

## 예외

- 연동 예외는 공통 부모(IntegrationException) 아래 연동별 단일 예외 하나만 두고 실패 유형은 status 유무와 응답 본문으로 구분함 — HTTP·타임아웃·네트워크 유형별 하위 예외는 기각
- 연동 예외의 요청 헤더·바디에는 접근 토큰(Authorization)과 client_secret이 원문으로 담기므로 로그 외부 전송이나 운영 배포 전에 마스킹을 먼저 도입해야 함 — 로컬 프로젝트라 에러 파악을 우선한 임시 결정

## 응답 처리

- 연동 계층은 응답을 envelope과 decimal 문자열까지 원형 그대로 반환하며 decimal 변환 등 가공은 상위 계층이 decimal.js로 수행함
- 성공 응답의 런타임 스키마 검증(zod)은 기각 — 의존성 추가와 인터페이스·스키마 이중 정의

## 토큰과 자격증명

- 연동 계층은 접근 토큰(accessToken)을 보관하지 않고 시세 메서드가 인자로 받으며 발급·보관·갱신은 상위 계층 책임 — 메모리 캐시안은 만료 처리 없는 상태 저장과 순환 의존으로 기각
- 연동 자격증명은 ConfigService로만 조회함 — process.env 직접 조회와 호출자 인자 전달안은 기각

## 테스트

- 연동 테스트는 client·api 계층 단위 테스트만 두고 실호출 스모크 테스트는 두지 않음 — 외부 의존

## 토스 명세 함정

- 토스 Open API 정본 명세는 openapi.json이며, 개발자센터 웹 문서는 API별 고정 앵커가 없어 링크는 Markdown 레퍼런스(api-reference/Apis/{Tag}Api.md#{operationId})를 씀
- 토큰 발급(POST /oauth2/token)은 form-urlencoded 본문을 받고 공통 envelope 없이 OAuth2 표준 형식으로 응답하며, 실패도 다른 형식({error, error_description})임
- 통화(Currency) 소비 코드는 다른 통화 값을 받아도 깨지지 않아야 함 — 명세가 현재 KRW·USD 외에도 알 수 없는 enum 값 허용을 요구함

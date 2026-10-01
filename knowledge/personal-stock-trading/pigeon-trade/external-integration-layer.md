---
description: 토스·TypeSafe AI 연동 클라이언트·API·예외를 추가하거나 고칠 때
---

# 외부 연동 계층

## 규칙

- 연동 계층은 서버가 내려준 데이터를 envelope까지 변환 없이 그대로 반환함 — 변환은 상위 계층 책임임
- 연동 계층은 토큰을 보관하지 않고 토스 시세 API 메서드(TossInvestMarketDataApi)가 accessToken을 인자로 받음

## 함정

- 연동 예외(`TossInvestException`·`TypeSafeAiException`)는 요청 헤더·바디를 마스킹 없이 원문으로 담아 Authorization 토큰·client_secret·API 키를 포함함 — 로그 외부 전송이나 운영 배포 전에 마스킹을 먼저 도입함

## 결정

- 토스 응답 zod 검증은 기각함 — 의존성 추가와 인터페이스·스키마 이중 정의가 필요함
- 토스 연동 예외(`TossInvestException`)의 HTTP·타임아웃·네트워크별 하위 예외는 기각함 — status 유무와 `responseBody.error.code`로 구분할 수 있음

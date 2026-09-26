---
description: MCP 서버 배포 형태나 도구 입력 스키마를 다룰 때
type: convention
---

# MCP 서버 구성

## 서버 제공 방식

- MCP 서버는 별도 stdio 프로세스 대신 기존 Nest 앱의 Streamable HTTP 엔드포인트로 제공하고 도구가 기존 서비스를 직접 호출함 — 배포 단위 1개와 서비스 검증 로직 재사용이 REST 응답 래핑을 다시 푸는 stdio안보다 유리함

## 도구 입력 스키마

- MCP 도구 입력 zod 스키마는 대응 요청 DTO의 class-validator 제약을 옮긴 이중 정의임 — 요청 DTO 제약을 바꾸면 도구 스키마도 함께 고쳐야 함

---
description: MCP 도구 응답 형식을 구현하거나 확인할 때
type: convention
---

# MCP 도구 응답 형식

## 성공·실패 응답

- MCP 도구 성공 응답은 REST 응답의 `data`와 같은 JSON(기존 응답 DTO 변환 결과, Decimal은 문자열)을 텍스트 콘텐츠 1건으로 반환함
- 서비스 예외는 `isError`와 `{ responseCode, message }` JSON 텍스트로 반환함 — 도구별 요약 텍스트 포매터안은 추가 코드 부담으로 기각
- MCP 도구 인자가 입력 스키마를 위반하면 MCP SDK(`@modelcontextprotocol/sdk`)가 핸들러 호출 전에 `isError`와 `Input validation error` 문구 텍스트로 응답함
- 입력 검증 실패 응답은 `{ responseCode, message }` JSON 형식이 아님 — 이 형식은 서비스 예외에만 적용됨

---
description: 메인 에이전트용 MCP 도구를 추가하거나 지갑 연결·호출 결과·오류를 다룰 때
---

# 에이전트 MCP 도구

## 적용 대상

- 메인 에이전트에 MCP 도구를 제공하는 pigeon-trade의 Nest 앱(AgentMcpController)

## 규칙

- 메인 에이전트용 MCP는 별도 stdio 프로세스가 아니라 기존 Nest 앱의 무상태 Streamable HTTP 엔드포인트(POST /api/mcp/wallets/:walletId)로 제공함
- MCP 도구는 지갑을 연결 URL의 walletId로 고정하고 도구 인자에는 walletId를 두지 않음 — 실행 루프가 `--mcp-config`로 지갑을 주입해 에이전트가 다른 지갑을 건드릴 수 없게 하며, 인자 방식은 프롬프트 전달·오입력 위험으로 기각함
- MCP 도구는 에이전트의 조회·주문 실행·판단 기록 동작만 노출함 — 지갑 생성·입금·종목 등록 같은 관리 동작은 REST 전용임
- MCP 판단 기록 도구(record_decision)는 type을 TRADE로 고정하고 action은 BUY·SELL·HOLD·WAIT만 받음 — Jev의 REGIME 기록은 실행 루프가 REST로 직접 호출함
- MCP 도구 성공 응답은 REST 응답 data와 같은 JSON 텍스트임 — 도구별 요약 텍스트 포매터안은 추가 코드 부담으로 기각함
- MCP 종목 목록 도구(list_stocks)는 `GET /api/stocks` 응답 항목과 같되 국면 필드(regime·regimeConfidence·regimeTimestamp·regimeModel)를 담지 않음 — 국면은 사용자 대시보드 참고용 정보임
- 서비스 예외는 isError와 `{ responseCode, message }` JSON으로 반환함

## 함정

- zod 입력 스키마 위반은 SDK가 핸들러 호출 전에 isError와 Input validation error 문구 텍스트로 응답함 — `{ responseCode, message }` 형식이 아님
- MCP 엔드포인트는 REST와 같이 인증이 없어 URL을 아는 누구나 해당 지갑으로 주문할 수 있음(의도된 단순화) — 앱을 외부에 노출하거나 에이전트를 여럿 운용하게 되면 토큰 헤더 인증이 필요함

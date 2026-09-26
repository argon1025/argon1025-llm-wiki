---
description: 메인 에이전트 헤드리스 실행의 인증·과금을 다룰 때
type: external
---

# Claude Code 헤드리스 실행 제약

## 제약

| 항목 | 제약 | 대응 |
| --- | --- | --- |
| 인증 | 구독 OAuth 토큰(`claude setup-token`, 유효 1년) 사용 시 구독 한도에서 차감 | `CLAUDE_CODE_OAUTH_TOKEN`으로 실행 |
| API 키 우선 | `ANTHROPIC_API_KEY`가 있으면 구독보다 우선해 API 과금 | 실행 환경에서 제거 |
| bare 모드 | `--bare`는 구독 토큰을 읽지 않고 CLAUDE.md·설정·훅 자동 로드를 끔 | 사용 금지, 에이전트 전용 작업 디렉터리로 로컬 설정 격리 |
| 사용 한도 | 구독의 5시간·주간 한도 적용 | Jev로 깨우는 횟수 최소화 |
| 권한 | `-p`는 권한 확인 창이 없음 | `--mcp-config`로 MCP 서버 지정, `--allowedTools`로 MCP 도구만 허용, `--disallowedTools`로 내장 도구 제거 |

## Agent SDK

- Agent SDK는 본인 구독 인증의 개인 사용 허용 여부가 문서에 없어 API 키 과금을 전제로 봄

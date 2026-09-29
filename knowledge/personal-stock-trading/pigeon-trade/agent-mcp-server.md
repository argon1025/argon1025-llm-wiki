---
description: MCP 도구 입력 스키마나 요청 DTO 검증 제약을 고칠 때
---

# MCP 도구 입력 스키마

## 규칙

- 요청 DTO의 class-validator 제약을 바꾸면 MCP 도구 입력 zod 스키마(AgentMcpService의 inputSchema)도 함께 고침 — 도구 스키마는 DTO 제약을 옮겨 적은 이중 정의임

---
description: 플러그인 스킬 문서·설명 문구를 쓰거나 버전을 올릴 때
---

# 플러그인 작성 규약

## 적용 대상

- my-claude-plugin-market 마켓플레이스와 그 안의 agent-wiki 플러그인

## 규칙

- agent-wiki 스킬 본문과 플러그인 문서는 '컨텍스트' 대신 '위키'라는 용어를 씀
- agent-wiki의 plugin.json·marketplace.json description과 루트 README 플러그인 표 설명은 '에이전트 위키'만 쓰고, 플러그인 README까지 이관 진행 상황·예정 기능 같은 현재 상태를 적지 않음
- 스킬 SKILL.md는 mktemp -d 결과를 셸 변수가 아니라 {tmp} 자리표시자로 옮겨 이후 명령에 쓰게 함 — Claude Code Bash 도구 호출 사이에는 셸 변수가 유지되지 않음
- 플러그인 버전을 올리는 PR은 그 플러그인 plugin.json의 version과 marketplace.json의 metadata.version을 함께 올림
- 플러그인별 version 원본은 marketplace.json이 아니라 각 플러그인의 plugin.json임

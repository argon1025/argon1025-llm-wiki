---
description: 개인판 agent-wiki 변경을 사내판 마켓플레이스로 미러링할 때
---

# 사내판 agent-wiki 미러링

## 적용 대상

- 개인판 my-claude-plugin-market의 agent-wiki 플러그인과 미등록 외부 시스템인 사내판 마켓플레이스의 plugins/agent-wiki

## 규칙

- 개인판 agent-wiki 변경은 사내판 마켓플레이스의 plugins/agent-wiki로 미러링함
- 미러링할 때 사내판 plugins/agent-wiki의 config.json·references/publish.md·README.md 3파일은 개인판 파일로 덮어쓰지 않음 — 세 파일은 사내판 고유 내용을 담음
- 개인판 변경이 고친 문구가 사내판 고유 3파일에도 있으면 그 문구만 개인판과 같게 고침

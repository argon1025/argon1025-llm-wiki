---
description: 위키 저장소 설정(config.json)이나 init의 저장소 선택을 고칠 때
---

# 위키 저장소 설정

## 규칙

- 에이전트 위키 저장소는 사용자가 고르지 않고 설정 파일(config.json)의 wiki.remote·wiki.baseBranch로 고정함 — init에 저장소 선택 질문(clone·새로 만들기)과 원격 없는 로컬 전용 모드를 두지 않아 단순화함
- agent-wiki 설정 파일(config.json)에는 공개 위키 URL만 두고 사내 위키 URL은 사내 플러그인이 별도로 선언함 — 마켓 레포(my-claude-plugin-market)가 PUBLIC 저장소임

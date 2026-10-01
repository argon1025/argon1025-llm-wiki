---
description: add 스킬의 호출 방식이나 위키 PR 게시·승인 흐름을 고칠 때
---

# add 스킬

## 규칙

- add 스킬(/agent-wiki:add)은 모델 호출을 허용함(disable-model-invocation 없음) — 세션 규칙 문구(generate_wiki_rules)가 고칠 내용이 있으면 /agent-wiki:add 반영을 제안하도록 안내함

## 결정

- add는 반영 초안을 사용자에게 승인받는 정지점을 두지 않고 위키 PR(wiki-add/{YYYYMMDD-HHMM})로 올림 — PR 머지가 초안 승인을 대신함

---
description: 위키 update 스킬의 머지 추출 범위·커서를 고치거나 점검할 때
---

# 위키 update 스킬

## 규칙

- 위키 update 스킬(/agent-wiki:update)은 지정한 도메인 하나(--domain)의 등록 레포 머지만 추출 대상으로 삼음 — 전체 도메인을 한 번에 넣으면 목록이 너무 많아짐
- update 추출 에이전트에는 해당 레포의 현재 지도인 책임(responsibilities)·호스트(hosts)·간선(deps)과 도메인 등록 레포 목록을 함께 넘김 — 넘기지 않으면 간선 대상(to)을 도메인 없이 오기하고 새 기능 단위의 책임 후보를 내지 못함

## 함정

- 커서를 최초 커밋으로 둬도 최초 커밋 자체는 반영되지 않음 — update는 커서를 제외한 `{cursor}..origin/{defaultBranch}` first-parent 커밋만 읽음

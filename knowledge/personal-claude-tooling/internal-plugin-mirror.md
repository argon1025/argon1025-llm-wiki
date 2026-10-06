---
description: 개인판 플러그인 공통 본문·버전을 바꾸거나 사내판 마켓플레이스로 미러링할 때
---

# 사내판 agent-wiki 미러링

## 적용 대상

- 개인판 my-claude-plugin-market의 agent-wiki 플러그인과 미등록 외부 시스템인 사내판 마켓플레이스의 plugins/agent-wiki
- 개인판 pr-workflow·plan-workflow·better-communication과 사내판 마켓플레이스의 대응 플러그인 devcenter-pr·devcenter-flow·devcenter-comms

## 규칙

- 개인판 agent-wiki 변경은 사내판 마켓플레이스의 plugins/agent-wiki로 미러링함
- 미러링할 때 사내판 plugins/agent-wiki의 config.json·references/publish.md·README.md 3파일은 개인판 파일로 덮어쓰지 않음 — 세 파일은 사내판 고유 내용을 담음
- 개인판 변경이 고친 문구가 사내판 고유 3파일에도 있으면 그 문구만 개인판과 같게 고침
- 사내판 고유 게시 절차 파일(publish.md)은 위키 원격이 Bitbucket인 사내판의 PR 게시 절차를 담음
- 개인판 게시 절차 파일(publish.md)은 위키 원격 호스트별 게시 수행 방안을 따로 안내하지 않음
- 개인판 agent-wiki와 사내판 plugins/agent-wiki는 두 위키의 등록 slug가 겹치지 않으면 같은 workspace.root(`~/.agent-wiki-workspace`)를 공유해도 됨 — 워크스페이스는 {slug} 폴더 단위 clone임
- 개인판 pr-workflow·plan-workflow·better-communication과 사내판 devcenter-pr·devcenter-flow·devcenter-comms는 사내 고유 파일만 다르고, 그 밖의 파일은 개인판이 원본이며 사내판은 rsync로 덮어씀
- 사내 고유 파일은 각 플러그인의 plugin.json·README.md와 devcenter-pr의 references/host.md, devcenter-flow의 config.json, devcenter-comms의 NOTICE임
- 개인판 pr-workflow·plan-workflow·better-communication에서 사내판으로 rsync되는 공통 본문(사내 고유 파일 밖의 파일)에는 플러그인 이름·사내 경로(.devcenter/…)·Bitbucket 도구명(bitbucket_*)·개인판 경로 상수(.ai-docs/…)를 넣지 않음 — 사내판이 공통 본문을 그대로 덮어써 환경 값이 섞이면 한쪽이 틀어짐
- 사내판 마켓플레이스의 플러그인 이름(devcenter-pr·devcenter-flow·devcenter-comms·agent-wiki)은 바꾸지 않고 유지함 — 사내 25개 레포 develop의 공유 .claude/settings.json이 이 이름을 enabledPlugins 키로 켜므로 바꾸면 25개 레포 설정 재배포와 marketplace renames 처리가 필요함
- 개인판 pr-workflow·plan-workflow·better-communication과 사내판 devcenter-pr·devcenter-flow·devcenter-comms는 대응 플러그인끼리 같은 버전을 씀
- 개인판 버전이 1.x 다음 pr-workflow 3.1.0·plan-workflow 3.2.0·better-communication 4.1.0으로 이어지는 것은 의도된 동작임 — 공통 버전을 사내 이력(devcenter-pr 3.0.1, devcenter-flow 3.0.1·사내 marketplace 항목 3.1.0, devcenter-comms 4.0.0)보다 높게 잡음

## 함정

- 개인판과 사내판의 wiki.baseRoot가 같은 경로(개인판 기본값 `~/.agent-wiki`)이면 나중에 끝난 쪽 위키를 다른 쪽 주입이 읽음 — 두 세션 시작 훅의 위키 동기화(sync_wiki.py)가 그 위키 사본을 각자의 원격(GitHub main·Bitbucket master)으로 remote set-url·강제 checkout해 경합함

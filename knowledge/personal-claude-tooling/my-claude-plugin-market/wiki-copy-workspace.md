---
description: 위키 사본·워크스페이스 동기화나 baseRoot 정리를 고칠 때
---

# 위키 사본과 워크스페이스

## 규칙

- 훅은 SessionStart의 source가 startup·resume·clear이면 위키 사본(wiki.baseRoot)을 원격 기준 브랜치로 강제 정리하고 compact이면 하지 않음 — /clear는 새 대화의 시작이고 compact는 진행 중 세션의 문맥 압축임
- 동기화 스크립트(sync_wiki.py·sync_register_repositories.py)는 origin이 기록된 remote와 달라도 멈추지 않고 remote set-url로 맞춤 — 두 폴더는 사본이라 보호할 로컬 상태가 없고 remote가 바뀐 경우도 흡수함
- 위키 기록(init 골격·register)은 `mktemp -d` 임시 clone에서 커밋·push함 — baseRoot는 세션 시작마다 강제 정리되므로 거기서 직접 쓰면 다른 세션이 열릴 때 수정이 지워짐
- 동기화 스크립트 두 개의 공통 git 실행 함수(git)는 파일마다 따로 두고 서로 import하지 않음 — 스크립트 하나 책임 하나·파일 간 의존 없음 원칙을 중복 제거보다 우선함
- 워크스페이스(workspace.root)는 사용 자유이며 세션 주입의 레포 지도(generate_repository_map)는 다른 에이전트가 덮어써 변경이 사라질 수 있다는 위험만 고지하고 기본 브랜치 기준 열람·최신화 같은 강제 지시는 두지 않음

## 함정

- baseRoot를 다른 용도의 기존 git 저장소로 지정하면 동기화가 remote set-url로 맞춘 뒤 그 저장소를 덮어씀 — 의도된 동작이며 대가임

## 결정

- 워크스페이스 일괄 최신화는 어떤 형태로도 도입하지 않음

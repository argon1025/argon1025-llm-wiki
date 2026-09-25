---
description: git worktree에서 앱을 로컬 기동할 때
type: procedure
---

# 로컬 앱 기동

## 트리거

- git worktree에서 앱을 기동하려 할 때

## 단계

- 메인 체크아웃의 env/.env.develop을 git worktree로 복사해야 기동됨 — env/.env.*는 gitignore 대상이라 worktree에는 존재하지 않음

## 분기와 실패

- git worktree에서 docker compose up -d 시 MySQL container_name(pigeon-trade-mysql)이 고정값이라 이름 충돌로 실패함
- 이미 떠 있는 컨테이너(포트 3310)를 모든 worktree가 개발 DB pigeon_trade·테스트 DB pigeon_trade_test로 함께 씀 — worktree의 migration:up도 이 공유 개발 DB에 적용됨

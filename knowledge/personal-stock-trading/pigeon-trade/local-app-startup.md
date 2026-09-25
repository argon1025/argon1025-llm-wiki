---
description: git worktree에서 앱을 로컬 기동할 때
type: procedure
---

# 로컬 앱 기동

## 트리거

- git worktree에서 앱을 기동하려 할 때

## 단계

- 메인 체크아웃의 env/.env.develop을 git worktree로 복사해야 기동됨 — env/.env.*는 gitignore 대상이라 worktree에는 존재하지 않음

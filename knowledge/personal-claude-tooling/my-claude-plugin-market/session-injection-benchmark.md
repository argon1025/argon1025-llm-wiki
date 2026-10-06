---
description: 세션 주입 벤치마크를 만들거나 `claude -p`로 돌릴 때
---

# 세션 주입 벤치마크

## 규칙

- 세션 주입 벤치마크의 하네스·과제·정답표·실행 기록은 레포 밖 `~/.agent-wiki-bench/session-injection/`에만 두고, 레포 기록에는 사내 레포명·문서명·API 경로·오류 코드를 쓰지 않음 — 이 레포가 PUBLIC임

## 함정

- `claude -p` 벤치마크에서 비교 주입만 남기려면 `--settings`의 enabledPlugins로 위키 계열 플러그인을 끄고 같은 파일의 hooks.SessionStart로 주입 JSON을 cat해야 함
- `claude -p` 벤치마크를 읽기 전용으로 돌리려면 `--permission-mode default`와 `--disallowedTools Edit Write NotebookEdit`를 명시해야 함 — 사용자 기본 권한 모드가 `auto`임

---
description: plan-workflow 세션 훅 설정 파일·plan 선행 읽기·feedback 근거를 다룰 때
---

# plan-workflow 작업 기록 규약

## 규칙

- plan-workflow의 feedback.md에서 같은 대상을 다르게 적은 기록이 있으면 항목 source:(그 사실을 정한 문서의 이름·판본, 이슈 키, 정책 문서 또는 사용자 확인 YYYY-MM-DD)의 판본·날짜가 최신 값을 가리는 근거가 됨
- plan.md의 선행 읽기 절은 세션 주입 문서를 절대 경로로 적음 — 실행은 새 세션에서 시작하므로 주입 형식에 기대지 않고도 열려야 함

## 함정

- 개인판·사내판 어느 판이든 세션 시작 훅이 플러그인 루트의 config.json(workspace.root)이나 plugin.json(name)을 읽지 못하면 작업 기록 규약이 주입되지 않음 — 훅이 두 파일을 읽다 난 python 예외를 삼키고 끝냄

---
description: plan-workflow 세션 훅 설정 파일·feedback 근거를 다룰 때
---

# plan-workflow 작업 기록 규약

## 규칙

- agent-wiki add가 입력으로 받는 plan-workflow의 feedback.md는 항목 source:(그 사실을 정한 문서의 이름·판본, 이슈 키, 정책 문서 또는 사용자 확인 YYYY-MM-DD)의 판본·날짜가 위키 기존 값과의 충돌을 가리는 근거가 됨
- feedback.md 항목에 source: 지목이 없으면 agent-wiki add가 그 항목을 코드 관찰로 취급함

## 함정

- 개인판·사내판 어느 판이든 세션 시작 훅이 플러그인 루트의 config.json(workspace.root)이나 plugin.json(name)을 읽지 못하면 작업 기록 규약이 주입되지 않음 — 훅이 두 파일을 읽다 난 python 예외를 삼키고 끝냄

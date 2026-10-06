---
description: 훅 리마인드 문구나 스킬 안내 명령의 플러그인 이름을 다룰 때
---

# 공통 본문의 플러그인 이름 지칭

## 규칙

- better-communication의 UserPromptSubmit 훅과 plan-workflow의 PostToolUse(ExitPlanMode) 훅은 리마인드 문구에서 플러그인 이름을 빼고 치환하지 않음 — 훅 실행마다 python으로 plugin.json을 파싱하는 비용을 피하고 정확한 명령은 세션 시작 주입문이 담음

## 함정

- pr-workflow 스킬이 안내하는 후속 명령(/<플러그인>:review·fix)이 잘못 조립되면 안내 문구만 틀리고 동작에는 영향이 없음 — pr-workflow는 훅이 없어 스킬을 부른 이름의 콜론 앞부분으로 명령을 조립함

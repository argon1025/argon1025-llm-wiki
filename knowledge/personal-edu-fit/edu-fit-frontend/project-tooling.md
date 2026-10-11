---
description: 브랜치·PR 대상을 정하거나 ESLint 제외·Prettier 포맷 검사를 다룰 때
---

# 프로젝트 도구 구성

## 규칙

- 기능 브랜치와 PR 대상은 `develop`이며 `main`은 배포 커밋만 받음
- `.ai-docs` 아래 작업 기록(plan.md·feedback.md)도 Prettier 형식으로 씀 — `npm run format:check`가 `.ai-docs` 마크다운도 검사함

## 함정

- ESLint 제외 목록(globalIgnores)의 shadcn 생성물 경로 `src/hooks/use-mobile.ts`는 파일이 없어도 의도된 동작이며 미사용 경로로 보고 지우지 않음 — shadcn sidebar를 추가할 때 함께 생성되는 훅임

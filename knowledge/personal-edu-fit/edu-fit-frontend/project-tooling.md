---
description: 브랜치·PR 대상, ESLint 제외·Prettier 검사, typecheck·import 정렬을 다룰 때
---

# 프로젝트 도구 구성

## 규칙

- 기능 브랜치와 PR 대상은 `develop`이며 `main`은 배포 커밋만 받음
- `.ai-docs` 아래 작업 기록(plan.md·feedback.md)도 Prettier 형식으로 씀 — `npm run format:check`가 `.ai-docs` 마크다운도 검사함
- React Bits 복사본 폴더(`react-bits`)는 ESLint 제외 목록(globalIgnores)에 둠 — 복사본을 원본 그대로 두고 프로젝트가 고친 곳만 `프로젝트 수정` 주석으로 표시함

## 함정

- `page.tsx` 같은 라우트 파일을 지운 뒤 개발 서버 없이 `npm run typecheck`를 돌리려면 먼저 `.next/dev/types`를 지움 — `next typegen`은 `.next/types`만 다시 만들고 tsconfig는 `.next/dev/types`도 포함해 남은 `validator.ts`가 지운 파일을 import해 실패할 수 있음
- 파일을 옮기거나 shared로 승격해 `@/` 경로가 바뀌면 import 순서(외부 패키지 → `@/` 경로순 → `../` → `./`)에 맞게 `@/` 묶음을 수동으로 다시 정렬함 — ESLint·Prettier가 import 순서를 강제하지 않음

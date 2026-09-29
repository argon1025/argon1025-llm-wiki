---
description: 빌드·린트·포맷·타입 검사 설정을 바꾸거나 검증 스크립트를 돌릴 때
---

# 빌드·린트 도구 설정

## 규칙

- Node 버전(.node-version)·Prettier 규칙(singleQuote, trailingComma all, printWidth 120)·패키지 매니저(npm)는 백엔드 pigeon-trade와 같은 값으로 둠
- Next 설정은 agentRules: false로 둠 — Next.js 16.3의 next dev·next build가 루트에 AGENTS.md·CLAUDE.md를 생성함
- 새로 생성된 shadcn 훅이 lint에 걸리면 훅을 고치지 않고 ESLint globalIgnores의 shadcn 생성물 목록에 그 경로를 추가함 — shadcn 생성물은 수정하지 않음
- shadcn 생성물을 추가하면 내용은 고치지 않고 npx prettier --write로 레포 Prettier 설정만 적용해 커밋함 — CLI 기본 포맷 그대로면 format:check에 걸림
- 작업 문서(.ai-docs/의 plan.md·feedback.md)의 표와 코드 블록도 Prettier 형식으로 씀 — .prettierignore가 .ai-docs/를 제외하지 않아 npm run format:check가 함께 검사함

## 함정

- next build 통과만으로는 린트 오류를 걸러내지 못해 npm run lint를 따로 실행함 — Next.js 16의 next build는 린트를 실행하지 않음
- 클린 체크아웃에서 tsc --noEmit만 실행하면 전역 타입(LayoutProps)을 찾지 못해 실패함 — 전역 타입은 next typegen이 생성하며 npm run typecheck가 typegen을 먼저 실행함

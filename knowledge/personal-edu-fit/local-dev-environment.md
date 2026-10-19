---
description: 로컬에서 프론트엔드·백엔드 포트를 정하거나 CORS origin을 맞출 때
---

# 로컬 포트와 CORS origin

## 적용 대상

- 프론트엔드 edu-fit-frontend, 백엔드 edu-fit-backend의 로컬 기동

## 규칙

- edu-fit-frontend는 로컬 포트 3101, edu-fit-backend는 3100을 씀 — pigeon-trade-dashboard(3001)·pigeon-trade(3000)와 동시에 기동하도록 겹치지 않게 정한 값임
- edu-fit-backend는 CORS에 edu-fit-frontend 개발·운영 서버 origin(`http://localhost:3101`)을 허용해야 함 — edu-fit-frontend가 브라우저에서 백엔드를 직접 호출함
- edu-fit-frontend worktree 화면 검증용 origin(예: `http://localhost:3109`)을 허용해야 하면 edu-fit-backend는 env 파일을 고치지 않고 기존 origin과 쉼표로 이은 `CORS_ORIGINS` 실행 환경 변수로 기동함 — `@nestjs/config`가 실행 환경 변수를 env 파일 값보다 우선함

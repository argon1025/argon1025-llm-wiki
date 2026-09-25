---
description: 환경변수 파일이나 새 실행 환경을 추가할 때
---

# 환경변수 설정 로드

## 규칙

- 환경변수 파일은 `env/` 폴더에 두고 `env/.env.${NODE_ENV ?? 'develop'}` 한 파일만 로드함 — 새 환경 추가 시 `env/.env.{환경명}` 생성과 `NODE_ENV` 지정 필요
- 실제 값 파일(`env/.env.*`)은 gitignore 대상이고 `env/.env.example`만 커밋함
- 현재 정의된 실행 환경은 develop 하나
- 토스 자격증명(클라이언트 ID·시크릿, `TOSS_INVEST_CLIENT_ID`·`TOSS_INVEST_CLIENT_SECRET`)은 `@nestjs/config`의 `ConfigService`로 조회함 — `process.env` 직접 조회와 호출자 인자 전달 방식은 기각함

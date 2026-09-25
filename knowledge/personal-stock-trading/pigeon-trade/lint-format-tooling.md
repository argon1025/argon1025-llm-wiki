---
description: 린트·포맷 도구나 그 스크립트를 바꿀 때
type: convention
---

# 린트·포맷 도구 구성

## 린트 도구 선택

- 린트는 ESLint + airbnb 대신 oxlint(type-aware) 규칙 강화로 운영함 — 원조 eslint-config-airbnb는 flat config 미지원·유지보수 중단이고 NestJS 12 스캐폴드가 oxlint를 기본 채택함(`.oxlintrc.json`)
- 기각한 대안은 커뮤니티판 eslint-config-airbnb-extended와 typescript-eslint strictTypeChecked이며 ESLint 재도입 비용이 기각 사유임

## 포맷 스크립트 함정

- test/ 폴더가 없는 동안 포맷 스크립트(`package.json#scripts.format`)의 --no-error-on-unmatched-pattern을 빼면 prettier가 test/**/*.ts 미매칭으로 exit 2를 반환하며, 린트 스크립트(lint)의 oxlint는 없는 test/ 경로를 오류 없이 무시함

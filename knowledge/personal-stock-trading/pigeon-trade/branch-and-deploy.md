---
description: 배포 경로를 정하거나 배포 때 마이그레이션을 실행할 때
type: procedure
---

# 브랜치와 배포 운영

## 배포 환경

- 운영 환경은 홈서버이며 Nest 공식 클라우드 배포(Mau, nest deploy)는 쓰지 않음

## 마이그레이션

- 배포 때 마이그레이션은 `npm run migration:up`으로 수동 실행함 — 앱 기동 시 자동 실행되지 않음

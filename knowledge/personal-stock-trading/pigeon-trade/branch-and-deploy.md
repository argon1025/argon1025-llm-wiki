---
description: 기능 브랜치·PR 대상이나 배포 경로를 정할 때
---

# 브랜치와 배포 운영

## 브랜치 운영

- 개발 기준 브랜치는 develop — 기능 브랜치 생성과 PR 대상 모두 develop으로 함
- master에는 배포 시점에 develop에서 머지한 배포 커밋만 들어감

## 배포 환경

- 운영 환경은 홈서버이며 Nest 공식 클라우드 배포(Mau, nest deploy)는 쓰지 않음

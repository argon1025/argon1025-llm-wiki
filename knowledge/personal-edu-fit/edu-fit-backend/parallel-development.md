---
description: 병렬 worktree에서 DB를 분리하거나 새 모듈을 추가할 때
---

# 병렬 개발

## 규칙

- worktree는 기본 체크아웃이 띄운 docker 컨테이너(`edu-fit-mysql`, 호스트 포트 3311)를 함께 씀
- worktree는 작업 단위 키별로 개발 DB `edu_fit_{키}`와 테스트 DB `edu_fit_{키}_test`를 만들어 개발·테스트 env 파일에 적음 — 같은 컨테이너 안에서 worktree끼리 데이터를 분리함

## 함정

- worktree에서 `docker compose up -d`를 다시 실행하면 같은 컨테이너 이름(`edu-fit-mysql`)으로 생성이 충돌함 — 기본 체크아웃이 같은 이름의 컨테이너를 이미 띄워 둠
- 병렬 브랜치가 각자 새 도메인 모듈을 `AppModule`에 등록(import 문 1줄·imports 항목 1줄)하면 같은 위치에 줄이 추가돼 develop 머지 때 충돌하며, 양쪽 줄을 모두 남기면 해소됨

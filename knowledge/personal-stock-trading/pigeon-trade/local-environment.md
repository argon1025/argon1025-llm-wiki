---
description: 로컬 DB·환경변수 파일을 설정하거나 worktree에서 앱을 기동할 때
---

# 로컬 개발 환경

## 규칙

- 로컬 MySQL 컨테이너는 호스트 포트 3310(`DB_PORT`)을 씀 — soop-dd(3309)와 포트가 충돌하지 않게 함

## 함정

- git worktree에서 앱을 기동하려면 메인 체크아웃의 `env/.env.develop`을 복사해야 함 — `env/.env.*`는 gitignore 대상이라 worktree에 없음
- worktree에서 `docker compose up -d`를 하면 컨테이너 이름 충돌로 실패함 — MySQL 컨테이너 이름(`container_name`)이 `pigeon-trade-mysql`로 고정임
- worktree의 `migration:up`은 메인 체크아웃·다른 worktree와 공유하는 개발 DB(`pigeon_trade`)에 적용됨 — 모든 worktree가 이미 떠 있는 한 MySQL 컨테이너(포트 3310)를 함께 씀
- 메인 체크아웃·worktree 앱이 같은 DB로 동시에 시세 동기화하면 현재가 리스너가 없는 이전 코드 앱이 먼저 저장한 1m 봉은 `stock.price`에 반영되지 않음 — 새 봉을 먼저 저장한 앱만 1m 저장 이벤트(`candle.1m.saved`)를 냄

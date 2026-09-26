---
description: pigeon-wallet 도메인 구현 세부를 다룰 때
type: convention
---

# pigeon-wallet 구현 컨벤션

## 명명

- pigeon-wallet의 API 경로·Swagger 설명·응답 메시지에는 '모의'·'가상'·'더미' 표현을 쓰지 않고 일반 지갑 표현만 씀 — 별칭 pigeon은 폴더·테이블·클래스 이름에만 둠

## 트레이드 정렬

- 트레이드 정렬은 opened_at 대신 구간 첫 주문 id 내림차순으로 함 — created_at이 소수 초 없는 datetime이라 같은 초에 열린 트레이드끼리 순서가 갈리지 않으며 주문 id는 체결 순서와 같음

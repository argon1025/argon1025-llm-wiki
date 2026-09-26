---
description: 도메인 폴더를 계층별로 나누거나 새 파일을 어디에 둘지 정할 때
type: convention
---

# 도메인 폴더 계층 구조

## 규칙

- market-data 도메인은 controller·service가 한 폴더에 섞여 가독성이 떨어져 controller/·service/·scheduler/ 계층별 폴더로 나눔 — 기능별 폴더안은 기각됨
- 적용 범위는 market-data뿐이며 stock 도메인은 기존 구조를 유지함

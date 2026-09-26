---
description: 도메인 폴더 구조를 정하거나 파일 위치를 판단할 때
type: convention
---

# 도메인 폴더 계층화

## 규칙

- 도메인 폴더는 controller/·service/·scheduler/ 계층별 폴더와 도메인 루트의 dto/·interface/·entity/·모듈 파일로 구성 — controller·service가 한 폴더에 섞이면 가독성이 떨어짐
- test 폴더도 위 구조를 그대로 미러링함
- 기능별 폴더안(기능 단위로 controller·service를 묶는 방식)은 기각
- stock 도메인은 기존 구조 유지

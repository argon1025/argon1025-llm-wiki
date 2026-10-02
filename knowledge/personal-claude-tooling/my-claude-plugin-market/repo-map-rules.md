---
description: 레포 지도 간선(deps)의 키 구성·같은 to 갱신이나 파급 조회 범위를 바꿀 때
---

# 레포 지도 간선 형식

## 규칙

- update·add의 지도 갱신에서 현재 간선에 같은 to가 이미 있으면 간선을 add하지 않고 그 desc에 사용처를 더한 replace로 둠 — verify_register_file.py가 같은 to 간선 두 개를 실패시킴

## 결정

- 간선(deps.json)은 to·desc 2키만 두고 계약 식별자 단위 파급 조회는 버림(의도된 동작)

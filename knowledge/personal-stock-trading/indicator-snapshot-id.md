---
description: 지표 스냅샷 ID 구성이나 구버전 스냅샷 조회 동작을 다룰 때
type: domain
---

# 지표 스냅샷 ID

## ID 형식

- 지표 스냅샷 ID는 `{isinCode}:{timeframe}:{봉 종료 UTC ISO}:{지표 버전}:{입력 해시 16자}` 합성키
- 입력 해시가 다르면 재수집 등으로 입력이 바뀐 것으로 보고 단건 조회에서 409로 알림

## 구버전 스냅샷 조회

- 지표 스냅샷 단건 조회는 현재 지표 버전(`INDICATOR_VERSION`)과 다른 버전 ID에 404 `INDICATOR_SNAPSHOT_NOT_FOUND`로 응답함 — 과거 버전 규칙으로 재계산할 수 없음, 구버전 조회의 최종 동작은 아직 정해지지 않음

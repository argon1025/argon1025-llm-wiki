---
description: 대시보드가 캔들 조회 API를 호출하는 방식이나 CORS 허용 origin을 다룰 때
type: policy
---

# 대시보드 API 접근

## 규칙

- pigeon-trade API의 CORS 허용 origin은 환경 변수 CORS_ORIGINS(쉼표 구분 목록)로 관리 — 비어 있거나 없으면 CORS를 켜지 않음
- 로컬 대시보드 origin http://localhost:3001을 허용함

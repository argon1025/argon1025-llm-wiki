---
description: 대시보드가 캔들 조회 API를 호출하는 방식이나 CORS 설정 여부를 다룰 때
type: policy
---

# 대시보드 API 접근

## 규칙

- 캔들 조회 API는 CORS를 설정하지 않음 — 대시보드가 Next.js 서버 측 fetch면 불필요, 브라우저 직접 호출 필요해지면 app.enableCors를 별도 작업으로 추가

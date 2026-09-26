---
description: 캔들 조회 API의 CORS 설정 여부를 판단할 때
type: convention
---

# 캔들 조회 API CORS 미설정

## 규칙

- 캔들 조회 API는 CORS를 설정하지 않음 — 대시보드가 Next.js 서버 측 fetch를 쓰는 동안은 불필요함
- 브라우저 직접 호출이 필요해지면 `app.enableCors`를 별도 작업으로 추가함

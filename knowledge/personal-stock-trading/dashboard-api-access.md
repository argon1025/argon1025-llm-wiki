---
description: 대시보드의 백엔드 호출, CORS·포트·API 주소를 설정할 때
---

# 대시보드 API 접근

## 적용 대상

- 브라우저에서 백엔드를 호출하는 pigeon-trade-dashboard, CORS와 포트를 설정하는 pigeon-trade

## 규칙

- pigeon-trade는 허용 origin 목록(CORS_ORIGINS, 쉼표 구분)에 넣은 origin만 CORS로 허용하고, 목록이 비어 있거나 없으면 CORS를 켜지 않음
- pigeon-trade의 CORS_ORIGINS에 대시보드 origin(`http://localhost:3001`)이 없으면 대시보드의 pigeon-trade 호출이 모두 실패함 — pigeon-trade-dashboard는 프록시 없이 브라우저에서 pigeon-trade를 직접 호출함
- pigeon-trade는 실행 포트(PORT)가 없으면 3000을 씀
- pigeon-trade-dashboard는 개발·운영 서버 모두 3001 포트를 씀 — pigeon-trade가 3000을 씀
- pigeon-trade-dashboard는 백엔드 주소를 NEXT_PUBLIC_API_BASE_URL로 받고, 없으면 `http://localhost:3000`을 씀

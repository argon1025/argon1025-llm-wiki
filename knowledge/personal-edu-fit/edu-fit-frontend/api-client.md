---
description: 백엔드 API 쿼리의 로드 상태를 판정하거나 백엔드 주소를 바꿀 때
---

# API 클라이언트

## 적용 대상

- edu-fit-frontend의 백엔드 API 호출과 TanStack Query 기반 로드 상태 판정

## 규칙

- 로드 상태 판정은 기존 데이터(data)가 있으면 isError보다 데이터를 먼저 판정함 — TanStack Query v5는 백그라운드 refetch 실패 시에도 isError가 켜져 기존 데이터를 가림

## 함정

- 백엔드 주소(NEXT_PUBLIC_API_BASE_URL) 값을 바꾸면 재빌드해야 함 — 빌드 시점에 클라이언트 번들로 인라인됨

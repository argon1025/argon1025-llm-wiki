---
description: 백엔드 API 쿼리의 로드 상태 판정·무효화나 백엔드 주소 변경을 다룰 때
---

# API 클라이언트

## 적용 대상

- edu-fit-frontend의 백엔드 API 호출과 TanStack Query 기반 로드 상태 판정

## 규칙

- 로드 상태 판정은 기존 데이터(data)가 있으면 isError보다 데이터를 먼저 판정함 — TanStack Query v5는 백그라운드 refetch 실패 시에도 isError가 켜져 기존 데이터를 가림
- 학생 수정 모달의 404(STUDENT_NOT_FOUND) 처리는 목록 쿼리(studentKeys.lists()·studentKeys.hasAny())만 무효화(invalidateQueries)하고 상세를 포함한 studentKeys.all은 무효화하지 않음 — 닫히는 동안 남은 상세 조회가 다시 404를 받아 같은 처리를 반복함

## 함정

- 백엔드 주소(NEXT_PUBLIC_API_BASE_URL) 값을 바꾸면 재빌드해야 함 — 빌드 시점에 클라이언트 번들로 인라인됨

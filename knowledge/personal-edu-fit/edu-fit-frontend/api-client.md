---
description: 백엔드 API 쿼리의 로드 상태 판정이나 재시도, 백엔드 주소를 다룰 때
---

# API 클라이언트

## 적용 대상

- edu-fit-frontend의 백엔드 API 호출과 TanStack Query 기반 로드 상태 판정

## 규칙

- 로드 상태 판정은 기존 데이터(data)가 있으면 isError보다 데이터를 먼저 판정함 — TanStack Query v5는 백그라운드 refetch 실패 시에도 isError가 켜져 기존 데이터를 가림
- 담당 학생 목록(TeacherStudents)은 total === 0일 때만 빈 상태를 그리고 items 0건·total > 0은 스켈레톤으로 그림 — 항목 수로 판정하면 넘친 쪽 보정 중 keepPreviousData가 이어받은 빈 결과로 '담당 학생이 없습니다'가 잠깐 보임
- 담당 학생 목록(TeacherStudents)은 placeholder 결과(isPlaceholderData)의 total 0을 빈 상태가 아닌 스켈레톤으로 그림 — keepPreviousData가 직전 결과의 0건을 이어받음
- 담당 학생 목록(TeacherStudents)과 학생 관리 목록(StudentManagement)은 data가 없고 isError이며 재조회 중이 아닐 때(!isFetching)만 조회 실패 카드를 그림 — '다시 시도' 뒤 재시도 동안 스켈레톤을 보임
- queryClient로 직접 조회할 때는 fetchQuery 대신 queryClient.query()를 씀 — 설치된 query-core 5.104.1에서 fetchQuery는 deprecated임

## 함정

- 백엔드 주소(NEXT_PUBLIC_API_BASE_URL) 값을 바꾸면 재빌드해야 함 — 빌드 시점에 클라이언트 번들로 인라인됨
- 저장 직전 재조회처럼 재시도하면 안 되는 조회는 queryClient.query()에도 retry: false를 명시함 — query()는 retry가 없을 때만 false로 두는데 전역 기본값(defaultOptions.queries.retry, 5xx·네트워크 오류 3회)이 먼저 합쳐짐

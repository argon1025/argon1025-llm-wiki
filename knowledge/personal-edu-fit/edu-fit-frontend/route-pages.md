---
description: 라우트 page.tsx를 추가하거나 동적 세그먼트·404 처리를 다룰 때
---

# 라우트 페이지

## 규칙

- useSearchParams를 쓰는 훅(useListParams·useTaskListParams)으로 목록 조건을 URL 쿼리에서 읽는 화면은 page.tsx에서 화면 컴포넌트를 Suspense로 감쌈 — 정적 프리렌더 페이지에서 Suspense 경계가 없으면 next build가 Missing Suspense boundary로 실패함
- 학생 요청 상세(/student/tasks/[taskId])와 수정(/student/tasks/[taskId]/edit)은 동적 세그먼트 이름을 taskId로 맞춤 — Next.js App Router는 같은 위치의 동적 세그먼트 이름이 다르면 빌드를 거부함

## 함정

- (app) 라우트 그룹의 동적 page에서 notFound()를 부르면 앱 셸 없는 Next 기본 404 화면이 HTTP 상태 코드 200으로 그려지는 것은 개발 서버·next start 모두 현재 정상 동작임 — 양의 정수가 아닌 taskId는 요청 없이 404 화면까지만 보장함

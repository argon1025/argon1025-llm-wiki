---
description: 학생 탐구 요청 등록·수정 폼의 수정 진입·저장·이탈 확인·중복 제출을 다룰 때
---

# 탐구 요청 폼

## 규칙

### 수정 화면

- 학생 탐구 요청 수정 화면(StudentTaskEditScreen)은 상세를 마운트 뒤 새로 받은 값(refetchOnMount 'always', isFetchedAfterMount)으로만 접수 여부를 판정함
- 수정 화면은 상세 조회의 창 포커스·재연결 재조회(refetchOnWindowFocus·refetchOnReconnect)를 끔 — 작성 중 재조회가 배정된 상태를 받으면 확인 없이 상세로 튕겨 입력을 잃음
- 작성 중 배정은 수정 저장 직전 상세 재조회(queryClient.query, staleTime 0, retry false)의 접수(RECEIVED) 판단(canStudentEdit)이 막음 — edu-fit-backend가 상태를 검사하지 않아 배정된 요청도 저장함
- 수정 저장 버튼은 trim 뒤 실제 보낼 필드가 있을 때(patchBody 키 존재, changed)만 켬 — 이탈 확인용 dirty(공백 변화 포함)와 판정을 나눠 공백만 바꾼 빈 PATCH가 나가지 않게 함

### 이탈 확인

- 탐구 요청 폼(useTaskForm)의 이탈 확인은 앱 셸 내부 링크·취소 버튼(확인 대화상자)과 새로고침·탭 닫기(브라우저 기본 확인)만 다룸
- 브라우저 뒤로가기·앞으로가기와 로그아웃·세션 만료에는 이탈 확인을 띄우지 않음(의도된 동작) — Next.js App Router에 이동 차단 API가 없음
- 뒤로가기 이탈 확인이 필요해지면 history 항목을 쌓고 popstate에서 확인하는 방식을 라우터 경합 검증과 함께 추가함
- beforeunload 이탈 확인은 세션(getSession())이 있을 때만 띄움 — 로그아웃·만료(logout·expireSession)는 clearSession() 뒤 window.location.replace로 문서를 옮겨 그 순간 세션이 빔
- 앱 셸 내부 링크(사이드바 메뉴·모바일 로고·헤더 뒤로 링크)를 가로채는 document 캡처 리스너는 preventDefault만 하고 stopPropagation은 안 함 — Next.js 16 Link는 자기 onClick 뒤 defaultPrevented면 이동하지 않음

## 함정

- 탐구 요청 폼(useTaskForm)은 onSubmit이 'done'(화면 이동 시작)을 돌려주면 처리 중 상태를 유지하고 이탈 확인만 끔(의도된 동작) — router.replace가 transition이라 이동 전 버튼이 다시 켜지면 등록 POST가 두 번 나감
- 링크 가로채기 리스너에서 전파를 막으면(stopPropagation) 모바일 사이드바 Link의 setOpenMobile(false)까지 막혀 Sheet가 열린 채 확인 대화상자가 겹침
- 수정 화면에서 작성 중 배정되면 저장할 때까지 알 수 없음(의도된 동작) — 창 포커스·재연결 재조회를 꺼 저장 직전 재조회에서만 배정을 확인함

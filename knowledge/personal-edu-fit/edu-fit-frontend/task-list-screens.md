---
description: 학생·원장 업무 목록·캘린더의 마감 강조와 필터·세그먼트를 다룰 때
---

# 업무 목록 화면

## 규칙

- 마감 긴급 강조(isUrgent, dueInfo)는 미완료 건의 D-0~D-3에만 붉은 굵은 글자와 알람 아이콘으로 씀
- 마감이 지난 미완료 건은 긴급 강조 없이 붉은 글자만 씀
- 마감일(formatDueDate)은 같은 해이면 MM.dd, 다른 해이면 yyyy.MM.dd로 표기함
- 상태색(TASK_STATUS_TONE)에는 붉은 계열을 쓰지 않음 — 긴급 강조가 붉은색이라 구분되지 않음
- 업무 목록·캘린더의 뷰(view=calendar)·세그먼트(segment=received|in-progress|completed|hidden)·쪽(page)·표시 월(month=YYYY-MM)은 URL 쿼리에 두고 기본값은 쿼리에서 지움
- 원장 탐구 컨설팅 관리의 세그먼트(전체·접수·진행 중·완료·숨김)는 목록 뷰에만 적용되고 캘린더 뷰는 학생·담당자 필터만 적용함 — 캘린더 API가 status·hidden을 받지 않기 때문임
- 원장 업무 조회 키(directorTaskKeys)는 루트 ['director-tasks'] 아래에 목록·숨김 여부·캘린더를 두어 숨기기 뒤 한 번에 무효화함
- 배정 후보·학생 선택지·학생 이름 조회 키는 ['director-tasks'] 루트 밖에 둠 — 필터 선택지가 다시 불리지 않게 함
- 원장 업무의 학생 필터(StudentFilter)는 비활성 학생을 포함한 서버 이름 검색(limit 20)으로 고름 — 졸업·퇴원 학생의 과거 업무도 목록에 남기 때문임
- URL로 들어온 선택 학생이 검색 결과에 없으면 학생 상세 조회로 이름을 받음
- 원장 업무의 학생·담당자 필터(useDirectorTaskFilters)는 URL 쿼리 student·assignee로 두고 1~4294967295 정수가 아니면 조건 없음으로 읽음 — 범위 밖 값은 백엔드가 400으로 거부함
- 배정 후보(활성 원장·선생)에 없는 담당자 ID는 후보를 받은 뒤 선택 없음으로 보고 조회에서 뺌
- 학생 업무 캘린더 뷰는 상태 세그먼트를 적용하지 않음 — 학생 캘린더 API(GET /api/student/tasks/calendar)에 status 필터가 없음

원장 업무 목록(DirectorTasksScreen)이 0건일 때의 빈 상태는 아래와 같음.

| 조건 | 빈 상태 |
|---|---|
| 학생·담당자 필터 적용 중 | '필터 초기화' 안내 |
| 필터 없음, 전체 세그먼트 | 숨김 1건 조회(hidden=true&limit=1)로 '업무 없음'과 '모두 숨김'을 가름 |
| 필터 없음, 접수·진행 중·완료·숨김 세그먼트 | 숨김 1건 조회 없이 세그먼트별 문구를 보임(의도된 동작) |

## 함정

- 공용 openRowProps는 keydown 대상이 행 자신인지 확인하지 않아 행 안 버튼에서 Enter를 누르면 행의 상세 이동이 실행됨 — 행 안 버튼(TaskHideButton)은 click과 keydown 전파를 모두 막음
- 배정 후보를 받기 전에는 URL의 담당자 ID(assignee)가 후보 확인 없이 첫 조회에 한 번 실림(의도된 동작)
- 학생 캘린더 조회(studentTaskQueries.calendar)에서 상세로 갔다가 뒤로가기로 돌아온 직후 재조회가 끝나기 전에 월을 옮기면 이전 달 흐림(keepPreviousData) 없이 빈 격자와 조회 중 표시만 보이며 원인은 아직 확인되지 않음

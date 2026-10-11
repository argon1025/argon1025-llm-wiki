---
description: 탐구 이력을 저장·조회·연결하거나 전공 적합성을 저장할 때
---

# 탐구 이력 분석

## 적용 대상

- 탐구 이력·연결·전공 적합성을 제공하는 edu-fit-backend, 학교급 노출·연결 중복·점수 범위를 판단하는 edu-fit-frontend

## 규칙

### 탐구 이력 조회

- edu-fit-backend는 탐구 이력을 학교급과 무관하게 모두 반환하고, 고등만 노출하는 설정은 edu-fit-frontend가 적용함
- 탐구 이력 목록(원장·선생·학생 공통)은 학교급(초등 → 중등 → 고등)·학년·학기·탐구 일자·ID 오름차순 정렬(compareInquiryRecords)로 반환함
- 학년·학기 미입력(null)은 같은 학교급·학년 안에서 값이 있는 이력 뒤에 둠
- 탐구 분석 목록 조회(탐구 이력·가져올 업무·탐구 연결·전공 적합성, 원장·선생·학생)는 목록 오프셋 페이지네이션 규약과 달리 페이지·전체 건수 없이 배열 전체를 반환함(의도된 동작)
- 탐구 이력의 학교급·학년·학기는 등록 시점 입력값으로 고정 저장해 학생 프로필과 동기화하지 않음
- 탐구 이력 담당자는 이름이 아니라 사용자 ID(assigneeId)로 저장하며, 응답의 assignee는 `{ id, name }`이고 미지정이거나 회원이 조회되지 않으면 null임

### 탐구 이력 등록·수정·삭제

- 탐구 이력 등록(POST /api/{director|teacher}/students/:studentId/inquiry-records)은 학교급·과목 및 유형·제목·탐구 일자(schoolLevel·subjectAndType·title·inquiryDate)를 필수로 받음
- 등록은 키워드·강점·보완점·후속 탐구 배열(keywords·strengths·improvements·followUps)도 필수로 받으며 빈 배열을 허용함
- 등록 본문으로 studentId·sourceTaskId는 받지 않음
- 탐구 축 필수·학교급별 학년 범위·빈 후속 탐구 제외는 edu-fit-frontend가 판단함
- 탐구 이력 등록·수정은 없거나 삭제된 탐구 축(axisId)을 404 INQUIRY_AXIS_NOT_FOUND, 없는 관련 자료 파일을 404 FILE_NOT_FOUND로 거부함
- 검증에 실패한 등록·수정은 이력을 저장하지 않음
- 탐구 이력 담당자(assigneeId)는 활성 여부와 무관하게 원장·선생 역할 회원이어야 하고 없거나 학생이면 404 ASSIGNEE_NOT_FOUND임
- 탐구 이력 등록·수정 요청 필드가 아래 범위를 벗어나면 400 INVALID_REQUEST임

| 필드 | 허용 범위 |
|---|---|
| 과목 및 유형·제목(subjectAndType·title) | 코드 포인트 1~255자, 공백만 불가 |
| 진로·후속 탐구 주제(career·topic) | 0~255자 |
| 요약·상세 분석·후속 탐구 내용(summary·analysis·content) | UTF-8 65,535 bytes 이하 |
| 학년(grade) | 1~255 |
| 학기(semester) | 1·2 |
| 탐구 일자(inquiryDate) | 1000~9999년의 실재하는 YYYY-MM-DD |

- 키워드·강점·보완점(keywords·strengths·improvements) 배열 원소에 짝 없는 서로게이트가 있으면 400 INVALID_REQUEST로 거부함
- 탐구 이력 수정(PATCH /api/{director|teacher}/inquiry-records/:recordId)은 전달한 필드만 바꿈
- 수정에서 null로 지울 수 있는 필드는 학년·학기·탐구 축·담당자·진로·요약·상세 분석(grade·semester·axisId·assigneeId·career·summary·analysis)뿐이며 나머지 필드의 null은 400임
- 실패한 수정은 함께 보낸 다른 필드도 바꾸지 않음
- 수정에서 followUps를 전달하면 후속 탐구를 전체 교체해 후속 탐구 ID가 새로 매겨짐
- 탐구 이력 updatedAt은 이력 컬럼 값이 실제로 바뀌거나 followUps를 전달하면 갱신됨
- 빈 본문이나 같은 값만 보낸 수정은 updatedAt을 바꾸지 않음
- 등록·가져오기 응답의 createdAt·updatedAt은 같은 초 단위 값임
- 탐구 이력 삭제(DELETE .../inquiry-records/:recordId)는 하드 삭제로 후속 탐구·관련 자료 연결·그 이력을 출발 또는 도착으로 잇는 탐구 연결을 함께 지움
- 삭제는 파일 자체를 남기고 응답 data는 null임
- 없거나 이미 삭제된 탐구 이력·탐구 연결 삭제는 404 INQUIRY_RECORD_NOT_FOUND·INQUIRY_LINK_NOT_FOUND임
- 같은 탐구 이력·탐구 연결을 동시에 삭제하면 하나만 200임

### 후속 탐구 관련 자료

- 후속 탐구 관련 자료(fileIds)는 등록이면 전체를, 수정이면 그 이력의 후속 탐구에 이미 연결된 파일을 뺀 새 파일만 요청자 업로드인지 확인함
- 타인이 올린 새 파일은 400 FILE_NOT_OWNED로 거부함
- 이미 연결된 타인 업로드 파일(가져온 이력의 승계 파일 포함)은 수정 때 유지하거나 다른 후속 탐구로 옮길 수 있음
- 같은 후속 탐구 안의 중복 파일 ID는 한 번만 연결하고 fileId 오름차순으로 응답함
- 파일 응답의 업로더 회원 식별자(uploaderId)는 원장·선생 응답에만 싣고 학생 본인 조회 응답에는 싣지 않음

### 완료 업무 가져오기

- 가져올 완료 업무 목록(GET .../students/:studentId/importable-tasks)은 학생의 철회되지 않은 완료 업무를 마감 기한 순으로 반환함
- 목록의 isImported는 그 업무를 원본(sourceTaskId)으로 가진 탐구 이력이 이미 있는지임
- 목록 항목의 제목은 산출물 제목이며 산출물이 없거나 제목이 null·빈 문자열·공백뿐이면 null이고, 과목 및 유형·마감 기한을 함께 실음
- 완료 업무 가져오기(POST .../students/:studentId/inquiry-records/import, body taskId)는 업무 상태를 보지 않음 — 완료 업무 한정은 목록에서 거름
- 완료 업무 가져오기는 경로 학생의 철회되지 않은 업무가 아니면 404 TASK_NOT_FOUND로 거부함
- 완료 업무 가져오기는 과목 및 유형·학교급·학년·학기·탐구 축·키워드·진로·후속 탐구를 탐구 이력에 복사하고, 피드백 메모를 요약으로, 현재 담당자를 담당자로, 마감일을 탐구 일자로, 업무 ID를 원본 업무 ID(sourceTaskId)로 저장함
- 직접 등록한 이력의 sourceTaskId는 null임
- 완료 업무 가져오기로 만드는 탐구 이력의 제목은 산출물 제목이며, 산출물이 없거나 산출물 제목이 null·빈 문자열·공백뿐이면 업무 과목 및 유형(subjectAndType)으로 대체함
- 완료 업무에서 가져온 탐구 이력은 등록 이후 원본 업무와 동기화하지 않음

### 탐구 연결

- 탐구 연결 목록은 생성순(ID 오름차순)으로 반환함
- 탐구 연결 생성은 출발·도착이 같은 이력이면 학생 확인 전에 400 INQUIRY_LINK_SELF로 거부함
- 출발·도착 중 하나라도 없거나 경로 학생의 이력이 아니면 404 INQUIRY_RECORD_NOT_FOUND로 거부함

### 전공 적합성

- 전공 적합성 저장(PUT .../students/:studentId/major-fits)은 학생 단위 전체 교체임
- 배열 순서가 표시 순서(sortOrder 1부터)이고 전공·역량 ID가 매번 새로 매겨지며 빈 배열이면 학생의 전공 적합성을 모두 지움
- 전공 이름(majorName)·역량 키워드(keyword)는 코드 포인트 1~100자(공백만 불가), 역량 의견(opinion)은 UTF-8 65,535 bytes 이하임
- 생략한 점수·의견은 null로 저장하고 id 필드를 보내면 400임
- 전공 적합성 조회(GET .../students/:studentId/major-fits)는 적합 전공을 표시 순서로 역량과 함께 반환함
- 조회는 평균·합계 같은 파생 값이 없고 데이터가 없으면 빈 배열임

### 선생·학생 접근

- 원장·선생 탐구 분석 API의 학생 경로는 없는 학생과 학생 프로필이 없는 회원을 404 STUDENT_NOT_FOUND로 거부함
- 선생의 탐구 이력·연결·전공 적합성 API는 담당 학생만 다룸
- 담당 범위 밖 학생·이력·연결은 존재 여부를 드러내지 않도록 없는 대상과 같은 404(STUDENT_NOT_FOUND·INQUIRY_RECORD_NOT_FOUND·INQUIRY_LINK_NOT_FOUND)로 거부함
- 선생의 가져올 업무 목록·가져오기는 담당 학생의 업무이면 다른 선생이 담당했거나 배정된 적 없는 업무도 대상으로 함
- 학생 본인 탐구 분석 조회(GET /api/student/{inquiry-records|inquiry-links|major-fits})는 읽기 전용으로 원장 조회와 같은 순서·값(관련 자료 uploaderId 제외)을 반환함
- 학생 본인 조회는 학생 프로필 존재를 확인하지 않아 프로필·데이터가 없으면 빈 배열 200임

## 코드값

원본: `AnalysisResponseCode` 전 3종

| 코드 | 응답 | 뜻 |
|---|---|---|
| `INQUIRY_RECORD_NOT_FOUND` | 404 | 존재하지 않는 탐구 이력, 연결 생성에서 경로 학생의 이력이 아닌 출발·도착, 선생 API에서 담당 범위 밖 학생의 이력 |
| `INQUIRY_LINK_NOT_FOUND` | 404 | 존재하지 않는 탐구 연결이거나 선생 API에서 담당 범위 밖 학생의 연결 |
| `INQUIRY_LINK_SELF` | 400 | 탐구 연결의 출발과 도착이 같은 이력 |

## 함정

- 탐구 이력 등록·수정 응답에는 text·varchar 필드의 짝 없는 서로게이트가 요청 원래 값으로, 이후 조회에는 U+FFFD로 나옴(의도된 단순화) — 저장 후 행을 다시 읽지 않고 저장 전 값으로 응답함
- 같은 업무를 다시(동시 요청 포함) 가져오면 탐구 이력이 하나 더 생김(의도된 동작) — 중복 가져오기 차단은 edu-fit-frontend가 isImported로 판단함
- 같은 (출발, 도착) 중복 탐구 연결도 생성됨 — edu-fit-backend가 중복과 학기 방향을 검증하지 않아 edu-fit-frontend가 판단함
- edu-fit-backend는 전공 적합성 역량 점수(score)가 int 범위 정수이면 0~10 밖 값(예 11)도 저장함 — 0~10 범위는 edu-fit-frontend가 판단함
- 전공 적합성의 역량 개수·빈 역량 행 제거·전공 이름 중복은 edu-fit-frontend가 판단함

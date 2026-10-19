---
description: 엔티티·테이블을 추가·변경하거나 소프트 삭제 조회·마이그레이션을 다룰 때
---

# 엔티티와 마이그레이션

## 규칙

### 엔티티 정의와 등록

- 새 엔티티는 src/{domain}/entity/{name}.entity.ts에 defineEntity로 정의해 자기 도메인 module의 MikroOrmModule.forFeature에 추가함
- 엔티티를 추가할 때 공용 database 설정(DatabaseModule)은 고치지 않음 — 앱은 autoLoadEntities로, CLI는 ./src/**/*.entity.ts glob으로 엔티티를 찾음
- 엔티티의 시각 필드는 luxon DateTime과 p.type(LuxonDateTimeType)(UTC SQL 문자열 저장)으로 선언하며 p.datetime()을 쓰지 않음
- 숫자 double 컬럼은 p.double().$type<number>().runtimeType('number')로 선언해 JS number로 다룸 — p.double()만 쓰면 TS 타입이 string | number로 추론됨
- 업무 마감일(task.due_date)은 luxon 변환 없이 응답에 싣고 범위 조회는 dueDate $gte·$lte 문자열 비교로 함 — p.date()가 MySQL date 컬럼을 YYYY-MM-DD 문자열로 매핑함

### 엔티티 참조

- 엔티티 사이 참조는 단방향으로 둠 — 역방향 컬렉션을 두면 엔티티 파일끼리 import해 lint import/no-cycle 오류가 남
- consulting 자식 테이블(TaskFile·TaskHidden과 이후 자식)은 p.manyToOne(Task) 단방향으로 두고 em.find(Child, { task: taskId })로 조회하며 Task에 p.oneToMany를 두지 않음
- 학생 목록·존재 확인·표시 학년 조회는 StudentProfile 기준으로 populate: ['user'] 조회함 — StudentProfile은 user를 PK·FK로 가진 단방향 1:1이고 User에 역방향 속성이 없음

### 소프트 삭제

- 소프트 삭제 테이블(subject·inquiry_axis·course·task)은 MikroORM 전역 filter가 없어 조회마다 deletedAt: null 조건을 직접 적음 — 빠뜨리면 삭제·철회된 행이 조회됨
- 수업(Lesson) 생성·수정·조회는 소속 수업 설정(Course)도 deletedAt: null 조건으로 확인함 — 확인하지 않으면 삭제된 수업 설정에 수업이 배정됨

### 테이블 설계

- inquiry_axis의 name·sort_order에는 unique를 두지 않음 — 이름 중복은 프론트 판단이고 순서 변경이 여러 행의 sort_order를 한 번에 바꾸며 삭제된 행이 기존 값을 유지함
- student_profile.grade 컬럼은 smallint unsigned이며 입력 상한 255는 컬럼 범위가 아닌 입력 범위임 — 학교급만 바꾸는 수정이 현재 표시 학년(255 초과 가능)을 지정 학년으로 저장하므로 tinyint unsigned이면 Out of range 500이 됨
- file의 uploader_id·storage_key에는 인덱스를 두지 않음 — 조회가 id IN (...)뿐임
- task_hidden은 별도 id 없이 (user_id, task_id) 복합 PK이며 userId를 task보다 먼저 선언해 PK 선두로 둠 — 목록 숨김 필터가 user_id로 찾음
- task 인덱스는 (student_id, due_date) 하나만 두며 담당자·상태 기준 조회를 추가하는 작업이 그 조회에 맞는 인덱스를 정함 — 기존 인덱스는 학생 본인 목록 정렬과 캘린더 범위 조회만 받침
- 히스토리 문구(task_history.message)는 TaskHistoryMessage로만 만들고, 이름을 둘 넘게 싣거나 user.name 컬럼을 넓히면 varchar(255) 상한을 다시 확인함 — 이름 두 개(각 최대 100자)를 담는 재배정 문구가 최대 207자임
- 댓글 첨부를 요청 순서로 응답해야 하면 연결 행(task_comment_file)에 자동 증가 id를 추가하고 id 순으로 정렬함 — 연결 행이 (comment_id, file_id) 복합 PK뿐이라 순서를 저장할 컬럼이 없음
- lesson 테이블은 FK 인덱스 course_id와 start_at 인덱스만 둠 — 원장 기간 조회는 start_at 범위로, 선생·학생 조회는 course.teacher_id·student_id 인덱스로 찾은 course_id로 하여 학원 규모에서 복합 인덱스 없이 충분함
- lesson_note는 FK 인덱스 lesson_id만 두고 복합 인덱스와 author_id 인덱스는 두지 않음 — 목록·최신 기록 조회가 모두 lesson_id로 찾고 수업 1건의 기록 수가 적으며 작성자 기준 조회가 없음
- 수업·수업 설정의 하드 삭제 경로를 만들면 같은 트랜잭션에서 수업 기록(lesson_note)을 먼저 지움 — lesson_note_lesson_id_foreign FK가 on delete 동작 없음(RESTRICT)이라 기록이 있는 수업의 하드 삭제를 DB가 거부함
- achievement_row는 FREE 행이면 school_level·grade·semester를 모두 null로, SEMESTER 행이면 셋을 모두 채워 저장함 — (table_id, school_level, grade, semester) unique에서 MySQL이 NULL을 서로 다른 값으로 봐 셋이 다 있는 행끼리만 중복을 막음
- 후속 탐구·역량의 표시 순서(sortOrder) 컬럼은 int로 둠 — 개수 상한이 edu-fit-frontend 판단이고 1MB 본문에 smallint 범위(65,535)를 넘는 항목이 들어갈 수 있음

## 절차

1. 엔티티와 DB가 어긋난 상태(develop 머지 직후 등)이면 먼저 migration:up을 실행함 — migrations.snapshot:false라 migration:create가 스냅샷 파일 없이 env/.env.develop DB의 실제 스키마와 엔티티를 비교함
2. migration:create가 만든 SQL이 자기 단위 테이블만 다루는지 확인한 뒤 커밋함 — 엔티티에 없는 테이블이 DB에 있으면 drop 문이 섞임
3. 엔티티와 DB의 차이만 미리 볼 때는 파일을 만들지 않는 npx mikro-orm schema:update --dump를 쓰며, 적용 후 차이가 없으면 Schema is up-to-date를 출력함

## 함정

- 엔티티 파일명이 .entity.ts로 끝나지 않으면 CLI가 찾지 못해 마이그레이션 diff에서 빠짐
- 미리보기에 npx mikro-orm migration:create --dump를 쓰지 않음 — --dump를 붙여도 마이그레이션 파일을 실제로 만듦
- 생성 직후 응답에 시각을 싣는 엔티티는 onCreate에서 DateTime.utc().startOf('second')로 초 단위로 잘라야 응답 값과 이후 조회 값이 같음 — LuxonDateTimeType 컬럼은 MySQL datetime(초 단위)이고 MySQL이 밀리초를 반올림해 저장함
- 수정 시각(onUpdate)과 수업 시작·종료 시각(toLessonTime)도 저장 전에 초 단위로 내림 — MySQL이 소수 초를 반올림해(09:00:00.600이 09:00:01) 응답 값과 이후 조회 값이 달라지고 같은 초 안의 시작·종료가 저장 후 같아질 수 있음
- 탐구 이력(InquiryRecord) 등록은 DateTime.utc().startOf('second') 한 값을 createdAt·updatedAt에 함께 넣음 — 두 필드의 onCreate는 한 flush 안에서 차례로 호출돼 그 사이에 초 경계를 넘으면 등록 응답의 두 시각이 달라짐
- 기간 조회의 from·to는 초로 내리지 않고 그대로 비교함 — LuxonDateTimeType 조건 값은 toSQL()의 밀리초까지 전달되고 MySQL이 datetime 컬럼과 소수 초 상수를 소수 초를 유지한 채 비교하므로 start_at = '09:00:00'은 >= '09:00:00.500'이 거짓임
- 학생 요청 수정(StudentTaskService.updateTask)에서 첨부 분류를 전달하면 task 행 값이 같아도 updatedAt을 직접 갱신함 — 첨부 연결은 task_file 행이라 Task의 onUpdate가 돌지 않음
- 원장·선생 산출물 저장(saveDeliverable)은 저장마다 산출물 updatedAt을 직접 갱신함 — 공유 파일·후속 탐구만 바뀌면 TaskDeliverable의 onUpdate가 돌지 않음
- 탐구 이력 수정(updateRecord)은 후속 탐구(followUps)를 전달하면 updatedAt을 직접 대입해 갱신함 — 후속 탐구만 바뀌면 InquiryRecord의 onUpdate가 돌지 않음
- Migration20261010185650을 task_assignment가 있던 예전 내용으로 적용한 DB는 다시 만듦 — 테이블을 지우는 마이그레이션이 없어 task_assignment가 남고 migration:create에 그 drop문이 섞임

## 결정

- 소프트 삭제에 MikroORM 전역 filter를 쓰지 않음 — 전역 filter는 External을 포함한 모든 em 조회에 보이지 않게 적용되어, 삭제 행까지 돌려줘야 하는 조회(SubjectExternalService.findByIds)가 매번 filter 해제를 챙겨야 하고 쿼리만 읽고 범위를 알 수 없음

---
description: e2e 테스트를 작성하거나 테스트 DB·시드·파일 저장 경로를 다룰 때
---

# e2e 테스트

## 규칙

- 공통 envelope가 아닌 응답(파일 스트림 등)은 fetch(`${e2e.baseUrl}/api/...`)로 직접 호출함
- 테스트 전용 컨트롤러는 spec 안에 선언해 E2eApp.create({ controllers })로 붙임
- worktree마다 env/.env.test의 DB_DATABASE를 자기 edu_fit_{키}_test로 둠 — spec 파일마다 schema.refresh()로 엔티티 테이블을 다시 만들므로 DB가 겹치면 서로의 데이터를 지움
- 과목·탐구 축을 참조하는 e2e는 spec 안에서 Subject·InquiryAxis 행을 직접 만듦 — 테스트 DB는 마이그레이션이 아닌 엔티티 기준 schema.refresh()로 만들어져 마이그레이션이 넣는 초기 수업 과목 5종·탐구 축 8종이 없음
- 프로필이 필요한 학생 회원은 POST /api/director/students로 등록하거나 StudentProfile을 직접 만듦
- 파일을 업로드하는 e2e spec은 E2eApp.create() 전에 process.env.FILE_STORAGE_PATH를 mkdtemp 임시 폴더로 바꾸고 afterAll에서 원래 값 복원과 폴더 삭제를 함 — 그러지 않으면 env/.env.test 경로에 업로드 실물이 쌓임
- HTTP 요청 없이 e2e.app.get(...)으로 꺼낸 서비스(External 등)를 직접 호출할 때는 RequestContext.create(e2e.orm.em, () => service.method(...))로 감쌈
- 요청 밖에서 서비스를 직접 만들어 호출할 때는 new MemberExternalService(orm.em.fork())처럼 fork한 EntityManager를 넘김
- 정렬 검증용 행은 em.create(Report, { createdAt, updatedAt, ... })로 시각을 지정해 DB에 직접 만듦 — em.create에 시각 값을 넘기면 LuxonDateTimeType 시각 필드의 onCreate가 덮어쓰지 않음
- e2e는 서버가 현재 시각(초)으로 계산해 응답에 싣는 값(로그인 expiresAt, 서명 downloadUrl의 expires·signature)을 요청 전에 잰 시각이나 다른 요청의 응답과 동등 비교하지 않음 — 동등 비교하면 초 경계에서 간헐 실패함
- 만료 시각은 요청 직전·직후 시각을 초로 내림한 before·after로 before+TTL ≤ 값 ≤ after+TTL 범위를 검사함
- 첨부(downloadUrl)가 담긴 응답을 다른 요청의 응답과 toEqual·toMatchObject로 비교할 때는 기대값을 withDownloadUrlPattern(...)으로 감쌈 — 서명 만료가 응답 시각(초)으로 정해져 두 요청이 다른 초에 처리되면 같은 첨부도 downloadUrl이 달라짐
- e2e 실패가 timeout이면 같은 spec을 단독 실행해 통과하는지로 공유 MySQL 부하와 테스트 결함을 구분함 — hookTimeout 30초는 beforeAll·beforeEach·afterAll에만 적용되고 testTimeout은 vitest 기본 5초 그대로임
- 초 경계 의존 여부는 setupFiles에서 luxon Settings.now를 1000배 가속(실제 1ms = 가상 1초)한 vitest 설정으로 spec을 돌려 확인함

## 함정

- E2eApp.createUser({ role: 'STUDENT' })로 만든 회원은 assertStudent·findStudentSummaries·학생 API에서 학생이 아닌 것으로 처리됨 — user 행만 만들고 student_profile은 만들지 않음
- 서비스는 전역 EntityManager를 주입받아 HTTP 요청 안에서는 요청 컨텍스트로 fork되지만 e2e·CLI처럼 요청 밖에서 호출하면 MikroORM이 전역 EntityManager 사용을 거부함
- 원장·선생 업무 상세(getTask)는 학생 프로필이 없으면, 목록(getTasks)은 업무가 참조하는 학생 회원이 없으면 데이터 정합성 Error로 500을 응답함 — 상세를 호출하는 e2e는 STUDENT 사용자를 만든 뒤 StudentProfile 행을 직접 만듦
- 같은 spec에서 원장 API로 등록하는 학생의 로그인 ID는 student{n}이 아닌 다른 접두어(예 managed{n})로 씀 — E2eApp.createUser가 로그인 ID를 지정하지 않으면 {역할 소문자}{순번}(예 student4)으로 만들어 user_login_id_unique 위반이 남
- 생성 응답의 updatedAt과 createdAt 동등 비교는 한 flush 안에서 초 경계를 넘으면 어긋나는 이론상 간헐 실패 여지가 있음 — 엔티티(TaskComment·Task·LessonNote·AchievementTable·Report)가 두 필드를 따로 onCreate로 채움
- 생성 응답 updatedAt·createdAt 비교의 간헐 실패는 테스트 코드로 결정적으로 막을 수 없음 — 엔티티가 생성 시각 하나를 두 필드에 함께 넣도록 운영 코드를 바꿔야 막힘
- 시계 가속 vitest 설정에서 만료된 토큰의 401 UNAUTHORIZED 테스트(GET /api/me) 1건이 실패하는 것은 가속의 부작용임(정상) — jsonwebtoken 만료 검증은 가속된 luxon 시각이 아닌 실제 Date.now()를 씀

## 결정

- 초 경계 간헐 실패를 vi.useFakeTimers로 앱 시계를 고정해 막지 않음 — 요청 간 시각 차이에서 생기는 실패를 숨김
- 첨부가 담긴 응답끼리 비교할 때 첨부를 []로 지우고 비교하지 않음 — 첨부 보존 여부를 검증하지 못함

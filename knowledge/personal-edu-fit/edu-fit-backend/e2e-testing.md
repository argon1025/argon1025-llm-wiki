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

## 함정

- E2eApp.createUser({ role: 'STUDENT' })로 만든 회원은 assertStudent·findStudentSummaries·학생 API에서 학생이 아닌 것으로 처리됨 — user 행만 만들고 student_profile은 만들지 않음
- 서비스는 전역 EntityManager를 주입받아 HTTP 요청 안에서는 요청 컨텍스트로 fork되지만 e2e·CLI처럼 요청 밖에서 호출하면 MikroORM이 전역 EntityManager 사용을 거부함

---
description: 도메인 모듈 경계·External 의존·DTO·interface 재사용·응답 코드 위치를 정할 때
---

# 도메인 모듈 경계

## 적용 대상

- edu-fit-backend의 도메인 모듈과 그 하위 controller·service·dto·interface 구성

## 규칙

- 도메인 경계는 인증, 회원, 수업 과목, 수업(수업 설정·수업·수업 기록), 리포트, 탐구 축, 컨설팅 업무, 탐구 분석, 성취도, 파일의 10개로 나눔
- FK는 같은 도메인 하위 테이블까지만 걸고 타 도메인 테이블은 ID로만 참조함 — 도메인 간 결합 방지
- 타 도메인 테이블의 표시 값은 애플리케이션에서 별도 조회해 조합함
- 도메인별 External 모듈은 External 서비스만 provide·export함
- External 서비스는 역할 서비스에 위임하지 않고 엔티티 매니저(EntityManager)로 직접 조회하며 다른 External 모듈을 import하지 않음 — 순환 의존 방지
- controller·service·dto·interface 아래에 역할(원장·선생·학생) 하위 폴더를 두고, 역할별 컨트롤러·서비스는 서로 독립임
- 요청·응답 DTO와 서비스 Option·Result interface는 도메인·역할·엔드포인트·계층(컨트롤러·서비스)마다 따로 선언하고, 기능이 완전히 같아도 다른 곳의 것을 재사용하지 않음 — 계층마다 바뀌는 원인이 다름
- 다른 엔드포인트 DTO를 `extends`·`Partial<>`·타입 별칭·공용 베이스 클래스로 가져오지 않고 목록 요청의 page·limit까지 엔드포인트마다 필드를 직접 선언함
- 서비스는 컨트롤러 DTO를 import하지 않으며, 컨트롤러가 요청 DTO 값을 서비스 Option으로 옮겨 담고 서비스 Result를 응답 DTO로 변환함
- 타 도메인 External 서비스가 돌려준 Result는 호출한 서비스 안에서 자기 interface로 변환해 쓰고 그대로 반환하거나 재export하지 않음
- enum(유니언 타입 포함)만 재사용을 허용하며, 여러 도메인이 쓰는 값은 공용 타입 모듈에, 도메인 고유 값은 소유 도메인에 둠
- 역할별 DTO 클래스명에는 역할 접두어를 붙임 — Swagger가 클래스명을 스키마 키로 써서 역할 간 동명 DTO가 덮어써짐
- 선생 담당 학생 판정은 수업 설정·업무 External 모듈만 import하는 전용 모듈 하나가 제공하고, 회원·리포트·탐구 분석·성취도의 선생 서비스가 씀 — 판정 중복과 순환 의존을 함께 피함
- 비밀번호 해시 함수(hashPassword·verifyPassword)는 특정 도메인이 아닌 공용 영역(common)에 둠 — auth·member·시드 CLI가 함께 쓰는 순수 함수라 도메인에 두면 다른 도메인이 External이 아닌 내부 파일을 import하게 됨
- 파일 다운로드 링크 서명 생성·검증(createFileUrlSignature·isValidFileUrlSignature)은 provider가 아닌 순수 함수로 두어 FileService(검증)와 FileExternalService(발급)가 함께 씀 — provider로 두면 External 모듈이 External 서비스 외 provider를 provide해야 함
- 공용 ResponseCode의 UNAUTHORIZED·FORBIDDEN·NOT_FOUND·INVALID_REQUEST는 도메인 응답 코드 파일에 다시 정의하지 않음 — 같은 code를 두 도메인이 정의하면 응답 message는 던진 쪽 항목을 따름
- External 서비스가 던지는 응답 코드(TEACHER_NOT_FOUND·ASSIGNEE_NOT_FOUND·INQUIRY_AXIS_NOT_FOUND·FILE_NOT_FOUND·FILE_NOT_OWNED 등)는 그 External 서비스가 있는 도메인 응답 코드 파일(예 MemberResponseCode)에만 정의하고 소비 도메인 응답 코드 파일에 다시 정의하지 않음
- 소비 도메인 컨트롤러는 External 서비스가 던지는 응답 코드를 Swagger 오류 응답(ApiErrorResponse)에 적을 때 그 External 서비스 도메인의 응답 코드 객체(예 FileResponseCode)를 import해 씀
- 같은 역할 안의 서비스끼리는 주입해 쓰고 역할 간 서비스 의존은 두지 않음(예 원장 업무 서비스 DirectorTaskService가 원장 댓글 서비스의 addComment를, 역할별 업무 서비스가 같은 역할 산출물 서비스의 findDeliverable을 호출)
- 역할 서비스들이 공유하는 도메인 규칙은 도메인 루트 함수(toTaskCommentContent·findLatestLessonNotes·assertAchievementItemRanges 등)로 둠 — 역할 서비스끼리 import하지 않고 규칙을 바꿀 때 그 함수만 고침
- 성취도 원장·선생·학생 서비스는 잠금·조회·응답 조합을 공유하지 않고 서비스마다 따로 둠
- 선생 리포트 서비스는 담당 범위 밖 학생 거부에 회원 응답 코드(MemberResponseCode)의 STUDENT_NOT_FOUND를 import해 던지고 리포트 응답 코드(ReportResponseCode)에 다시 정의하지 않음
- 수업·수업 기록·리포트 서비스는 응답에 싣는 학생·선생·작성자 ID를 모아 회원 이름 조회(MemberExternalService.findUsersByIds)를 요청당 한 번 호출함
- 수업·수업 기록·리포트 서비스는 회원 이름 조회 결과에 참조한 회원이 없으면 기본값 없이 데이터 정합성 Error(500)를 던짐(requireName·requireAuthorName) — 회원은 하드 삭제 경로가 없어 참조 대상이 없으면 오류로 봄

## 함정

- defineResponseCodes를 거치지 않은 code를 실은 HttpException은 ApiExceptionFilter가 status별 공용 코드로 바꿔 응답하는 것이 정상 — 내부 문자열 노출을 막음
- 도메인 Module을 AppModule에서 빼면 그 도메인 External 모듈만 import한 소비 모듈의 External 조회가 실패함 — External 모듈(SubjectExternalModule·ConsultingExternalModule·CourseExternalModule)은 엔티티를 등록하지 않고 AppModule에 있는 도메인 Module의 MikroOrmModule.forFeature에 의존함

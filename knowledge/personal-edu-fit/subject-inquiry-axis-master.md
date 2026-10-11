---
description: 수업 과목·탐구 축 마스터를 추가·수정·삭제·정렬·조회하거나 탐구 축을 지정할 때
---

# 수업 과목·탐구 축 마스터

## 적용 대상

- 과목·탐구 축 마스터 API를 제공하는 edu-fit-backend와 이를 호출하는 edu-fit-frontend

## 규칙

- 수업 과목 삭제는 참조 여부와 무관하게 항상 소프트 삭제(deletedAt 기록)이며, 삭제된 과목은 신규 선택 목록에서만 빠지고 기존 수업 설정·선생 담당 과목의 참조와 이름 조회는 유지됨
- 탐구 축 삭제는 소프트 삭제이며, 삭제된 축을 참조하는 업무·탐구 이력은 조회 응답에서 축 없음(미분류)으로 표기하고 타 도메인 데이터는 갱신하지 않음
- 탐구 축은 순서 값으로 테크트리 세로축 순서를 관리하고, 수업 과목은 이름순으로 정렬함
- 활성 탐구 축 목록 조회는 전 역할 공통 API로 제공하고, 생성·수정·삭제·순서 변경만 원장 API에 둠
- 과목·탐구 축 삭제 전 참조 건수를 알려주는 삭제 영향 조회 API는 없음
- 수업 과목 이름(name)은 코드 포인트 기준 1~50자만 검증해 위반이면 400 INVALID_REQUEST이고 trim과 공백만 있는 이름 거부는 하지 않음
- 초기 수업 과목 5종은 마이그레이션으로만 들어가 ID 1 수학, 2 영어, 3 과학, 4 국어, 5 사회를 받으며 deletedAt은 null임
- 원장 수업 과목 목록(GET /api/director/subjects)은 목록 오프셋 페이지네이션 규약과 달리 페이지·전체 건수 없이 삭제되지 않은 과목을 이름 오름차순(같은 이름은 ID순) {id, name} 배열로 줌(의도된 동작)
- 원장 수업 과목 수정·삭제(PATCH·DELETE /api/director/subjects/:subjectId)는 삭제되지 않은 과목만 대상으로 하며 없는 과목과 이미 삭제된 과목은 404 SUBJECT_NOT_FOUND(존재하지 않는 과목입니다.)임
- 원장 수업 과목 수정·삭제(/api/director/subjects/:subjectId)의 subjectId가 정수 문자열이 아니면 400 INVALID_REQUEST이고, 음수 정수는 형식 검증을 통과해 404 SUBJECT_NOT_FOUND임
- 수업 과목 삭제 성공은 data null로 응답하고 삭제 과목의 복구·수정 API는 없음
- 활성 탐구 축 목록(GET /api/inquiry-axes)은 전 역할(원장·선생·학생)이 호출하며 목록 오프셋 페이지네이션 규약과 달리 삭제되지 않은 축 전체를 sortOrder 오름차순(같으면 id 오름차순) 배열로 반환함(의도된 동작)
- 원장 탐구 축 관리 API(/api/director/inquiry-axes)는 목록(GET, 페이지네이션 없는 배열)·추가(POST, 201)·순서 변경(PUT /order)·이름 수정(PATCH /:axisId)·소프트 삭제(DELETE /:axisId)를 제공함
- 원장 탐구 축 관리 API는 DIRECTOR 전용이라 선생·학생이 호출하면 403 FORBIDDEN임
- 탐구 축 이름(name)은 코드 포인트 기준 1~50자이고 공백만으로 된 이름은 허용되지 않으며 위반이면 400 INVALID_REQUEST임
- 탐구 축 추가(POST)는 삭제되지 않은 축 중 가장 큰 sortOrder에 1을 더한 값(활성 축이 없으면 1)으로 마지막 순서에 붙이며 삭제된 축의 sortOrder는 계산에 쓰지 않음
- 탐구 축 순서 변경(PUT /api/director/inquiry-axes/order)은 활성 탐구 축 ID 전체(axisIds)를 중복 없이 한 번씩 원하는 순서로 받음
- 탐구 축 순서 변경에서 누락·중복·미존재(삭제된 축 포함) ID가 있으면 'axisIds: 활성 탐구 축 ID 전체를 중복 없이 한 번씩 전달해야 합니다.' 메시지의 400 INVALID_REQUEST임
- 탐구 축 순서 변경이 통과하면 받은 순서대로 sortOrder를 1부터 다시 매기고 그 목록을 반환함
- 탐구 축 삭제는 deletedAt만 채우고 성공 시 data는 null이며 다른 축의 sortOrder는 바꾸지 않음
- 없는 탐구 축과 이미 삭제된 탐구 축의 이름 수정·삭제는 모두 404 INQUIRY_AXIS_NOT_FOUND이고 숫자가 아닌 axisId는 400 INVALID_REQUEST임
- edu-fit-backend는 탐구 축을 새로 지정하는 쓰기 요청의 축(axisId)이 없거나 삭제된 축이면 404 INQUIRY_AXIS_NOT_FOUND로 거부함
- 탐구 축 테이블(inquiry_axis)의 초기 8종은 국어 1·수학 2·영어 3·사회 4·과학 5·예체능 6·동아리 7·진로 8(이름 뒤 sort_order)로 생성 마이그레이션에서 삽입됨

## 함정

- 수업 과목 삭제는 기획서 CLS-26·BR-32의 '미참조 시 실제 삭제, 참조 시 비활성'과 달리 참조 여부와 무관하게 항상 소프트 삭제임(의도된 동작)
- 수업 과목 이름 중복은 edu-fit-backend가 막지 않아(unique 제약 없음) 같은 이름의 과목도 추가됨 — 중복 판단은 edu-fit-frontend 몫임
- 소프트 삭제된 수업 과목의 이름은 새 과목이 다시 쓸 수 있음
- 탐구 축 이름 중복도 edu-fit-backend가 확인하지 않고 edu-fit-frontend가 판단함
- 삭제된 탐구 축의 미분류는 시스템 행이 아니라 axisId null이므로 edu-fit-frontend가 '미분류 활동' 행으로 렌더함

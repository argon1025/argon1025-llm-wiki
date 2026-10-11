---
description: 다른 도메인 External 서비스로 과목·탐구 축·첨부 파일을 검사할 때
---

# External 서비스 호출

## 규칙

- 과목을 쓰는 도메인은 과목 없음 코드(SUBJECT_NOT_FOUND)를 자기 응답 코드에 다시 정의하지 않음 — 과목 활성 확인(SubjectExternalService.assertActive)이 과목 도메인의 SUBJECT_NOT_FOUND를 던짐
- 탐구 축을 새로 지정하는 쓰기 API는 축 활성 확인(InquiryAxisExternalService.assertActive)으로 삭제되지 않은 축인지 검사함
- 응답에 싣는 탐구 축 ID는 모아 활성 축 조회(InquiryAxisExternalService.findActiveIds)를 한 번 호출하고, 결과 집합에 없는 ID를 axisId null(미분류)로 바꿈
- 삭제된 탐구 축을 참조하는 업무·탐구 이력 응답의 미분류(axisId null) 변환은 탐구 축 도메인(InquiryAxisExternalService)이 아니라 응답을 만드는 소비 도메인(컨설팅 업무·탐구 분석)이 수행함
- 도메인 요청으로 새로 연결하는 파일 ID(fileIds)는 업로더 확인(FileExternalService.assertUploadedBy)으로 요청자가 업로드한 파일인지 검사함
- 같은 대상에 이미 연결된 파일 ID는 호출 도메인이 빼고 업로더 확인(assertUploadedBy)에 넘김 — FileExternalService는 파일의 연결 대상을 몰라 기존 연결 여부를 가리지 못함

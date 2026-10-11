---
description: 역할별 API를 나누거나 응답·페이지네이션·시각 규약을 정할 때
---

# API 규약

## 적용 대상

- API를 제공하는 edu-fit-backend, 응답을 받아 화면에 표시하는 edu-fit-frontend

## 규칙

- 계정당 역할은 원장·선생·학생 중 하나임
- API는 원장·선생·학생 역할별로 분리해 제공하고, 인증·파일·탐구 축 조회처럼 역할 차이가 없는 기능만 공통 API로 둠
- 기능이 같아도 역할이 다르면 API를 역할별로 따로 둠 — 역할마다 바뀌는 원인이 다름
- 역할 범위 밖 리소스 접근은 존재하지 않는 리소스와 같은 응답으로 거부해 존재 여부를 노출하지 않음
- 조회는 되나 쓰기 권한이 없을 때(타인 작성 기록 수정 등)만 권한 없음 응답으로 거부함
- 목록 조회는 페이지 번호·페이지 크기로 나누는 오프셋 페이지네이션이며 전체 건수를 함께 돌려줌
- DB와 edu-fit-backend는 모든 시각을 UTC로 다루고, 현지 시각 표시 변환은 edu-fit-frontend가 함
- edu-fit-backend 응답은 공통 envelope(success·responseCode·message·data)이며 실패 시 data는 null임
- edu-fit-backend는 실패 응답을 HTTP 4xx/5xx 상태와 success=false로 함께 내려줌
- edu-fit-frontend는 성공을 HTTP 상태가 아니라 응답 envelope(ApiResponse)의 success로 판정함
- edu-fit-frontend는 GET 요청(apiGet)에서 값이 undefined인 쿼리 파라미터를 키째 빼고 보냄 — edu-fit-backend가 정의되지 않은 쿼리 키를 400으로 거부함
- edu-fit-backend는 요청 회원의 역할이 @Roles로 지정한 역할과 다르면 403 FORBIDDEN으로 거부하고, 토큰이 없으면 역할 확인(RolesGuard) 전에 401 UNAUTHORIZED로 거부함
- edu-fit-backend는 @Roles가 없는 비공개 API를 인증된 모든 역할에 허용함

## 코드값

원본: `Role` 전 3종

| 코드 | 뜻 |
|---|---|
| `DIRECTOR` | 원장 |
| `TEACHER` | 선생 |
| `STUDENT` | 학생 |

원본: `ResponseCode` — 일부

| 코드 | HTTP 상태 | 기본 메시지 |
|---|---|---|
| `UNAUTHORIZED` | 401 | 인증이 필요합니다. |
| `FORBIDDEN` | 403 | 접근 권한이 없습니다. |

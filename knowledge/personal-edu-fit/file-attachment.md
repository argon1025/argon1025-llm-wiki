---
description: 파일을 업로드하거나 도메인에 첨부·해제·다운로드할 때
---

# 파일 첨부

## 적용 대상

- 파일을 업로드·저장하고 서명 URL을 발급하는 edu-fit-backend, 파일 ID로 첨부를 연결하고 서명 URL로 내려받는 edu-fit-frontend

## 규칙

- 파일 업로드(POST /api/files)는 전 역할이 multipart/form-data의 file 필드로 파일 1개(최대 20MB=20×1024×1024 bytes, 확장자 제한 없음)를 올려 파일 ID(fileId)를 받음
- 도메인 요청은 업로드로 받은 파일 ID를 fileIds로 보내 첨부로 연결함
- 파일 다운로드는 도메인 조회 응답이 파일마다 싣는 단기 서명 URL로 인증 헤더 없이 받음 — 링크 다운로드는 Bearer 헤더를 실을 수 없음
- 파일 다운로드(GET /api/files/:fileId/download)는 인증 헤더 없이 쿼리 expires(만료 epoch 초)·signature(HMAC-SHA256 소문자 hex 64자)가 맞을 때만 파일 스트림을 내려줌
- 다운로드 권한은 응답을 만드는 도메인(downloadUrl을 응답에 싣는 쪽)이 판정함 — 파일 도메인은 파일의 소속을 모름
- 다운로드 서명(만료 포함)이 맞지 않으면 없는 fileId도 404가 아닌 403 FILE_URL_INVALID로 응답하고, 서명이 맞을 때만 없는 행을 404 FILE_NOT_FOUND로 응답함 — 서명을 파일 조회보다 먼저 확인하는 의도된 동작임
- downloadUrl은 env FILE_URL_TTL_SECONDS(기본 600초) 동안 유효함 — 만료 전까지 누구나 인증 없이 쓸 수 있어 링크가 유출되면 그동안 내려받을 수 있음
- 도메인 요청으로 새로 연결하는 파일 ID는 요청자가 업로드한 파일이어야 함 — 타인 파일을 연결해 서명 URL을 얻는 우회를 막음
- 첨부 해제는 도메인 연결 행만 지우고 파일 행·실물 파일은 남김 — 같은 파일 ID를 여러 곳이 공유해도 안전함
- 완료 업무 가져오기는 후속 탐구 첨부를 파일 ID 공유로 승계함
- 아래 조건의 파일 업로드·다운로드 요청은 400 INVALID_REQUEST로 거부함

| 요청 | 조건 |
|---|---|
| 업로드 | 파일명 또는 MIME 타입이 255자(코드 포인트 기준)를 넘음 — 실물 저장 전에 거부 |
| 업로드 | 파일 외 텍스트 필드가 10개를 넘거나 값이 1024bytes 이상 |
| 다운로드 | expires 누락·비정수, signature 누락·빈 문자열, 정수가 아닌 fileId, 정의되지 않은 쿼리 |

## 코드값

원본: `FileResponseCode` 전 5종

| 코드 | 응답 | 뜻 |
|---|---|---|
| `FILE_REQUIRED` | 400 | 업로드할 파일이 없음(file 파트 없음·JSON 요청·빈 파일명) |
| `FILE_TOO_LARGE` | 413 | 업로드 파일이 20MB를 넘음 |
| `FILE_NOT_OWNED` | 400 | 다른 회원이 업로드한 파일을 첨부로 연결하려 함 |
| `FILE_URL_INVALID` | 403 | 다운로드 링크가 만료됐거나 서명이 맞지 않음 |
| `FILE_NOT_FOUND` | 404 | 존재하지 않는 파일 |

## 함정

- 어느 도메인에도 연결되지 않은 업로드 파일은 정리되지 않고 남는 것이 정상 — 고아 파일 정리 기능은 두지 않음
- 업로드에서 file 외 필드명의 파일이나 file 파일 2개 이상은 400 INVALID_REQUEST로 응답하며 FILE_REQUIRED가 아님
- 도메인 응답 첨부의 downloadUrl은 API 서버 origin 없는 상대 경로(/api/files/:fileId/download?expires=..&signature=..)임 — edu-fit-frontend가 API base URL을 앞에 붙여 요청해야 함

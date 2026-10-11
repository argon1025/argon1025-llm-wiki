---
description: 파일 저장소·S3 전환이나 multer 업로드·다운로드 응답을 구현할 때
---

# 파일 저장소

## 규칙

- 파일 저장소는 로컬 디스크로 시작하되 S3 전환이 가능하도록 저장 전략 인터페이스로 분리함
- FileService는 실물을 먼저 저장한 뒤 file 행을 만듦 — 중간 실패 시 다운로드 불가 행 대신 고아 실물만 남음
- 원본 파일명을 응답 헤더에 실을 때는 다운로드 Content-Disposition(filename*=UTF-8'')처럼 RFC 5987 퍼센트 인코딩(encodeRfc5987)을 거침 — 파일명의 CR·LF·큰따옴표가 헤더로 새지 않게 막는 곳이 이 인코딩뿐임
- 업로드(FileController.upload)가 multer 메모리 저장으로 최대 20MB를 버퍼에 담은 뒤 Readable.from(buffer)로 FileStorage.save에 넘기는 것은 의도된 단순화임
- 동시 업로드로 메모리가 문제되면 업로드를 FileStorage.save로 바로 쓰는 multer 저장 엔진으로 바꿈
- FileInterceptor를 쓰는 핸들러는 옵션에 defParamCharset: 'utf8'을 넘김 — multer는 multipart 파트 헤더의 파일명을 기본 latin1로 해석해 한글 파일명이 깨짐
- FileInterceptor의 limits.fileSize 초과는 업로드 핸들러에서 FileInterceptor 앞에 둔 FileTooLargeInterceptor가 FILE_TOO_LARGE로 바꿔 던짐 — Nest는 응답 코드 없는 PayloadTooLargeException을 던지고 전역 필터가 이를 413 INTERNAL_SERVER_ERROR로 응답함

## 함정

- file 행은 있으나 실물 파일이 없으면 LocalFileStorage.read가 ENOENT로 실패해 다운로드가 500 INTERNAL_SERVER_ERROR로 응답함(의도된 동작) — 실물 누락은 저장소 장애로 보아 별도 응답 코드를 두지 않음
- busboy는 텍스트 필드 값이 limits.fieldSize와 같은 길이에 도달하면 잘린 값으로 봄 — multer fieldSize N은 N bytes 미만 값만 받고, 필드 개수 fields N은 N개까지 받음
- 업로드 mimeType(FileController.upload)은 busboy가 파트 Content-Type의 ASCII 토큰 문자로만 type/subtype을 만들고 형식이 아니면 text/plain으로 둠 — ASCII 외 문자가 들어올 수 없음

---
description: 요청 본문 크기·객체 속성 수 상한이나 본문 파싱·검증 오류 응답을 다룰 때
---

# 요청 크기 상한

## 적용 대상

- edu-fit-backend의 JSON·urlencoded 요청 본문, 요청 본문·쿼리·경로 값의 객체, 검증 오류 응답

## 규칙

- JSON 요청 본문 상한은 1MB(1,048,576 bytes)임
- urlencoded 요청 본문 상한은 Express 기본 100kb(102,400 bytes)이고 파라미터는 1,000개까지임
- JSON·urlencoded 본문 파싱 오류(크기 초과, urlencoded 파라미터 1,000개 초과, 미지원 charset·Content-Encoding, 손상된 압축 본문, 잘못된 JSON)는 413·415·500이 아닌 400 INVALID_REQUEST로 응답함
- 상한을 넘거나 잘못된 본문은 비인증 요청이라도 400 INVALID_REQUEST로 거부됨 — 본문 파싱이 인증 Guard보다 먼저 실행됨
- 본문 파싱 오류 응답에도 허용 출처의 CORS 헤더(Access-Control-Allow-Origin)가 실림
- 요청 본문·쿼리·경로 값 안의 객체(중첩 포함) 하나의 속성이 100개를 넘으면 DTO 변환 전에 400 INVALID_REQUEST(`{경로}: object must have at most 100 properties`, 예 `body: ...`)로 거부됨
- 임의 키를 받는 맵형 요청 필드가 생기면 객체 속성 100개 상한을 다시 정해야 함
- 검증 오류 응답 message는 필드 사유를 선언 순서로 앞의 20건까지만 ' / '로 이어 싣고 나머지는 '외 N건'으로 줄임

## 결정

- 본문 파싱 오류에 413·415 전용 응답 코드를 두지 않음 — 같은 TEXT 상한 초과 입력이 JSON 이스케이프 크기에 따라 DTO 검증(400)과 본문 상한으로 갈려도 같은 코드가 되고 multer 텍스트 필드 상한 초과(400)와 맞음

---
description: 요청 DTO의 길이·날짜·boolean·page 상한·PATCH null 검증을 추가하거나 고칠 때
---

# 요청 DTO 검증

## 규칙

- 목록 쿼리 DTO(DirectorGetStudentsRequest)는 page와 limit 상한을 모두 둠 — page 상한이 없으면 offset (page−1)×limit이 MySQL LIMIT 범위를 넘거나(page=1000000000000000000) 지수 표기(page=1e300)로 SQL에 들어가 SyntaxErrorException 500이 됨
- PATCH 요청(DirectorUpdateStudentRequest)에서 null을 허용하지 않는 필드는 @ValidateIf((_, value) => value !== undefined)로 두고, null로 값을 지우는 필드만 @IsOptional()을 씀 — 전역 ValidationPipe에서 @IsOptional()은 null도 검증 없이 통과시킴
- 쿼리 DTO(DirectorGetStudentsRequest)의 boolean은 'true'·'false'만 바꾸는 @Transform 뒤 @IsBoolean()으로 검증함 — 전역 ValidationPipe가 암묵 변환을 하지 않고 @Type(() => Boolean)은 'false'를 true로 바꿈
- 쿼리 DTO의 숫자는 @Type(() => Number)로 변환함 — 전역 ValidationPipe가 암묵 변환을 하지 않음
- 요청 문자열 길이 검증은 @Length·@MaxLength 대신 코드 포인트 기준 @Matches(/^[\s\S]{min,max}$/u)를 DTO에 인라인으로 두고 위반은 400 INVALID_REQUEST로 응답함 — @Length는 변형 선택자(U+FE0F·U+FE0E)를 빼고 세어 검증을 통과한 값이 MySQL varchar 길이를 넘어 500이 됨
- DTO 밖에서 들어오는 문자열(FileService.upload)은 저장 전 Array.from(value).length로 코드 포인트 길이를 확인해 상한을 넘으면 400 INVALID_REQUEST로 거부함 — 확인하지 않으면 varchar 상한을 넘는 값이 Data too long 500이 됨
- UTF-8 바이트 상한 검사(isWithinDetailsBytes)는 DTO 안에서 Buffer.byteLength(value, 'utf8')로 함 — mysql2가 저장하는 값(짝 없는 서로게이트는 U+FFFD 3 bytes)과 같음
- 날짜 문자열 검증(DirectorCreateCourseRequest)은 IsISO8601({ strict: true })와 /^[1-9]\d{3}-\d{2}-\d{2}$/u 정규식을 함께 써 MySQL date 지원 범위의 YYYY-MM-DD만 받음

## 함정

- class-validator @IsByteLength는 encodeURI로 바이트를 세어 짝 없는 서로게이트 입력에 URIError를 던져 500이 됨
- IsISO8601({ strict: true })만으로는 시각이 붙은 값도 통과함

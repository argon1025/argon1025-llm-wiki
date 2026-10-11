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
- text 컬럼 UTF-8 바이트 상한 검사는 class-validator ByteLength 대신 Buffer.byteLength(value, 'utf8') 기반 ValidateBy(maxUtf8ByteLength)로 함 — mysql2가 저장하는 값(짝 없는 서로게이트는 U+FFFD 3 bytes)과 같음
- 날짜 문자열 검증(DirectorCreateCourseRequest)은 IsISO8601({ strict: true })와 /^[1-9]\d{3}-\d{2}-\d{2}$/u 정규식을 함께 써 MySQL date 지원 범위의 YYYY-MM-DD만 받음
- 요청 DTO의 중첩 객체 필드(DirectorAssignTaskRequest)는 @ValidateNested()에 @IsObject()를 함께 둠 — @ValidateNested()만 두면 배열(빈 배열 포함)이 통과함
- 객체 배열 요청 필드(TeacherSaveDeliverableRequest, TeacherSaveMajorFitsRequest)는 @IsObject({ each: true })를 함께 둠 — ValidateNested가 원소 자리의 배열을 다시 펼쳐 검증해 통과시킴
- UTC 시각 입력 DTO(DirectorCreateLessonRequest)는 형식 정규식(@Matches)과 @IsISO8601({ strict: true })를 함께 둠 — 정규식만 두면 2026-02-30·24:00·60초가 통과해 luxon DateTime.fromISO가 24:00을 다음 날 00:00으로 해석함
- UTC 시각 입력 DTO의 형식 정규식은 Z 접미만 받아 @IsISO8601이 허용하는 오프셋 입력을 거름 — 오프셋 입력은 UTC 변환 뒤 연도가 MySQL datetime 범위를 벗어날 수 있음
- 요청 DTO에 새로 두는 id 필드(회원·축·담당자·업무·파일·이력 ID: fileIds 원소, studentId 쿼리 등)는 @Min(1)·@Max(4_294_967_295)로 int unsigned 범위를 검증함 — 상한이 없으면 지수 표기 값(1e300)이 SQL에 들어감
- 자유 문자열을 MySQL 8 JSON 컬럼에 저장하는 요청 필드(산출물 keywords 등)는 짝 없는 서로게이트를 \P{Cs} 정규식으로 400 INVALID_REQUEST 거부함 — JSON 컬럼은 짝 없는 서로게이트를 이스케이프한 JSON 텍스트를 ERROR 3140으로 거부해 500이 되며 varchar·text 컬럼은 해당하지 않음

## 함정

- class-validator @IsByteLength는 encodeURI로 바이트를 세어 짝 없는 서로게이트 입력에 URIError를 던져 500이 됨
- IsISO8601({ strict: true })만으로는 시각이 붙은 값도 통과함
- @Param('...', ParseIntPipe) 경로 ID가 int unsigned 범위 밖이면 20자리 이하 정수(4294967296)는 통과해 조회 결과 없음(404)이 되고 21자리 이상 정수는 1e+21 표기가 되어 400 INVALID_REQUEST가 됨 — 전역 ValidationPipe(transform: true)가 ParseIntPipe보다 먼저 Number로 바꿈
- 요청 DTO 한 속성에 @Matches를 두 번 달면 둘 다 실패할 때(빈 문자열) 400 메시지에 첫 번째 사유만 남음 — class-validator가 제약을 이름(matches)으로 묶음

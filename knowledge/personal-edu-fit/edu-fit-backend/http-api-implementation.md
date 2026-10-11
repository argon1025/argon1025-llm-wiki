---
description: HTTP API 컨트롤러·응답 DTO·Swagger를 구현하거나 앱 설정을 고칠 때
---

# HTTP API 구현

## 규칙

- 리소스를 생성하지 않는 POST API(로그인 AuthController.login 등)는 @HttpCode(HttpStatus.OK)로 200을 응답함 — 201은 생성 API에만 적용함
- 역할별 컨트롤러는 AuthModule을 import하지 않고 클래스에 @Roles('DIRECTOR')를 붙이고 핸들러에서 @CurrentUser() user: AuthUser를 받음 — AuthModule이 JwtAuthGuard·RolesGuard를 APP_GUARD로 전역 등록함
- @Public()은 로그인·헬스·파일 다운로드에만 붙임
- 응답 DTO의 luxon DateTime 필드는 @Type(() => String)과 원본 obj의 DateTime을 toUTC().toISO()로 바꾸는 @Transform을 함께 달아야 함 — @Type 없이 두면 plainToInstance가 DateTime을 빈 생성자로 복제하다 실패함
- 전역 접두어 /api·CORS처럼 앱 인스턴스에 거는 HTTP 설정은 configureApp(app)에 둠 — 앱 기동(bootstrap)에만 두면 e2e 앱에 반영되지 않음
- 컨트롤러마다 @ApiBearerAuth()를 붙이지 않음 — Swagger 설정(setupSwagger)이 addBearerAuth()·addSecurityRequirements('bearer')로 모든 API에 Bearer 인증을 전역 지정함
- data null로 응답하는 엔드포인트는 @ApiOkResponse({ description: '성공 시 data는 null' })로 문서화하고 공용 Swagger 데코레이터는 고치지 않음 — 공용 ApiSuccessResponse가 응답 모델 클래스를 요구함
- 파일 스트림 응답은 @Res()로 직접 pipe하지 않고 new StreamableFile(stream, { type, length, disposition })을 반환함 — 전역 ApiResponseInterceptor가 StreamableFile 반환값만 envelope로 감싸지 않고 통과시킴
- 스트림을 반환하는 핸들러는 @Res({ passthrough: true }) 응답의 close 이벤트에서 스트림을 destroy하고 이미 끊긴 응답이면 바로 닫음 — Nest ExpressAdapter는 StreamableFile 스트림을 pipe만 하고 연결이 끊겨도 원본 스트림을 닫지 않음
- 업무 컨트롤러에서 @Get('calendar')는 @Get(':taskId')보다 먼저 선언함 — 뒤에 두면 calendar가 업무 ID로 해석되어 ParseIntPipe가 400을 냄

## 함정

- @Public() 핸들러에서 @CurrentUser()를 쓰면 401 UNAUTHORIZED로 응답함 — @Public() API에는 request.user가 없음
- JSON 요청 본문이 Express 기본 body-parser 한도 100KB를 넘으면 PayloadTooLargeError를 전역 ApiExceptionFilter가 500 INTERNAL_SERVER_ERROR로 응답함 — 65535 bytes 이하 수업 설정 세부사항(details)이라도 이스케이프가 많아 본문이 100KB를 넘으면 400이 아닌 500이 됨

---
description: 로그인·계정 발급·비밀번호 정책·회원 비활성화를 다룰 때
---

# 계정 인증

## 적용 대상

- 계정을 발급·인증하는 edu-fit-backend와 그 토큰·인증 실패 응답을 받는 edu-fit-frontend

## 규칙

- 로그인은 ID/비밀번호이며 자체 회원 가입은 없고 학생·선생 계정은 원장이 발급함
- 인증은 JWT Bearer access 토큰 하나(수명 12시간)이며 refresh 토큰은 아직 없음
- 인증이 필요한 API는 Authorization: Bearer access 토큰(HS256, 클레임 sub=회원 ID 문자열·role·exp)으로 호출함
- edu-fit-frontend는 토큰을 붙인 요청이 응답 코드 UNAUTHORIZED로 실패하면 세션 만료로 처리함 — 로그인 실패는 401 INVALID_CREDENTIALS로 구분됨
- 인증이 필요한 API는 토큰 없음·형식 오류·서명 불일치·만료·없는 회원·비활성 회원이면 모두 401 UNAUTHORIZED로 응답함
- edu-fit-backend는 매 요청 회원의 활성 여부를 DB에서 다시 조회해 비활성화된 회원의 기존 토큰도 401 UNAUTHORIZED로 즉시 거부함
- 요청의 역할은 토큰의 role 클레임이 아니라 DB의 현재 역할을 씀 — 발급 뒤 바뀐 역할이 바로 반영됨
- 로그인(POST /api/auth/login)은 로그인 ID(loginId)·비밀번호(password)를 받아 accessToken·expiresAt(UTC ISO 8601, 발급 12시간 뒤)·user(id, name, role)를 200으로 응답함
- 로그인에서 없는 ID·비밀번호 불일치·비활성 계정은 구분하지 않고 모두 401 INVALID_CREDENTIALS로 응답함
- 로그인 요청의 필드 누락·문자열 아님·정의되지 않은 필드는 400 INVALID_REQUEST로 응답함
- 내 정보 조회(GET /api/me)는 로그인한 회원의 id·loginId·name·role을 응답함
- 회원 비밀번호는 user.password_hash에 scrypt:{salt hex}:{hash hex} 형식(scrypt, 16바이트 salt, 64바이트 키)으로 저장함
- 계정 발급·재지정과 원장 시드가 password_hash에 같은 형식을 저장해야 로그인이 동작함
- 비활성 계정 로그인 실패는 ID·비밀번호 불일치와 같은 실패 응답으로 돌려줘 구분하지 않음
- 비밀번호 정책은 문자 종류 제한 없이 최소 8자이며 발급·재지정에 동일 적용함
- 회원 비활성화는 수업·업무·탐구 이력 데이터를 보존하고 로그인만 차단하며, 비활성 회원 재활성화를 허용함

## 코드값

원본: `UserStatus` 전 2종

| 코드 | 뜻 |
|---|---|
| `ACTIVE` | 활성 — 로그인과 토큰 사용이 가능함 |
| `INACTIVE` | 비활성 — 데이터는 보존하고 로그인과 기존 토큰 사용을 차단함 |

원본: `AuthResponseCode` 전 1종

| 코드 | HTTP 상태 | 기본 메시지 |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | ID 또는 비밀번호가 올바르지 않습니다. |

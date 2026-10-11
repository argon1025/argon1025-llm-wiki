---
description: 원장 초기 계정을 만들거나 비밀번호 해시·로그인 회원 조회를 구현할 때
---

# 원장 초기 계정과 비밀번호 해시

## 규칙

- 원장 초기 계정은 시드 CLI로 1회 수동 생성하며 비밀번호를 파일·env에 남기지 않음
- 비밀번호 해시는 node:crypto scrypt로 salt를 포함해 저장하며 외부 해시 패키지를 쓰지 않음

## 절차

- 운영 환경의 원장 시드는 `npm run build` 후 `NODE_ENV=production node dist/member/cli/seed-director.js`로 실행함 — seed:director 스크립트의 tsx는 devDependency라 운영에 없음

## 함정

- 원장 시드(seedDirector)를 다른 로그인 ID로 다시 실행하면 원장 계정이 하나 더 생김(의도된 동작) — 원장 계정 수를 제한하지 않으며 원장 초기 계정의 1회 생성은 코드로 막지 않는 운영 절차임
- 로그인(AuthService.login)은 없는 로그인 ID이면 scrypt 검증 없이 바로 실패해 응답 시간으로 ID 존재 여부를 추정할 여지가 있음(의도된 동작) — 계정 열거 방어가 필요해지면 없는 ID에도 더미 해시 검증을 수행함

## 결정

- 인증용 회원 조회(MemberExternalService.findAuthUser)는 조회 키별 메서드로 나누지 않고 { userId } | { loginId } 조건 하나로 로그인·매 요청 활성 확인·내 정보를 모두 처리하며 비활성 회원도 반환함

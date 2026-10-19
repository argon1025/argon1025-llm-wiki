---
description: 보호 화면의 로그인·역할 가드나 데이터 조회 방식, 앱 셸 세션 판정을 다룰 때
---

# 인증·역할 가드

## 규칙

- 인증·역할 가드는 서버 레이아웃이나 proxy가 아닌 클라이언트(AppShell)에서 판정함 — 세션(access 토큰·사용자)이 localStorage에만 있고 useSession의 서버 스냅샷이 null임
- 앱 셸(AppShell)은 가드 판정 전에는 보호 화면을 그리지 않음 — 그리면 다른 역할 화면이 잠깐 보임
- 역할 접두어 경로(/director·/teacher·/student)는 해당 역할만 접근하고 접두어 없는 경로(/)는 모든 역할이 접근함 — 클라이언트 가드(canAccess)는 화면 가림용이고 최종 권한 판단은 서버 응답(403·404)임
- 보호 화면의 데이터는 서버 컴포넌트에서 조회하지 않고 localStorage 토큰을 쓰는 클라이언트 요청으로만 다룸 — 서버 컴포넌트 페이지의 출력은 클라이언트 가드가 children을 그리지 않아도 RSC payload에 실림
- 앱 셸 가드 effect는 useSession() 값이 아니라 getSession()으로 저장소를 직접 읽어 판정함 — 하이드레이션 첫 커밋의 passive effect는 서버 스냅샷(null) 렌더의 클로저를 봐 로그인 사용자도 새로고침 때 /login이나 /로 튕김
- 앱 셸 가드 effect는 session을 deps에 넣지 않아 세션 없음 판정을 마운트·경로·me 역할 변경 때만 함 — 만료·로그아웃(expireSession·logout)의 clearSession() 알림에 다시 돌면 가드의 router.replace가 뒤따르는 window.location.replace와 경쟁해, 버려질 문서의 로그인 화면이 만료 안내 플래그를 소비하고 새 문서에서 만료 토스트가 사라짐

## 함정

- 역할 가드는 실제 page가 있는 경로에만 적용됨 — Next.js App Router는 없는 경로를 (app) 레이아웃을 거치지 않고 루트 not-found로 처리함
- 발급 뒤 역할이 바뀌었거나 localStorage를 고친 경우 /api/me 도착 전 저장된 역할의 메뉴가 잠깐 보이는 것은 감수한 동작임 — 그 사이 요청은 서버가 403·404로 막음

## 결정

- /api/me 응답을 기다린 뒤 셸을 그리는 안은 기각함 — 새로고침마다 빈 화면과 실패 화면이 필요함

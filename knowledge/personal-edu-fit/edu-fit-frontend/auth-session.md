---
description: 로그인 폼, 세션 만료·종료, 다른 탭 세션 변경을 다룰 때
---

# 인증 세션

## 규칙

- 로그인 폼(LoginForm)은 method="post"를 둠 — 폼이 하이드레이션 전부터 보여 그 사이 제출이 기본값 GET으로 나가면 비밀번호가 주소·기록·서버 로그에 남음
- 로그인 폼(LoginForm)의 진입 애니메이션은 Motion initial 대신 CSS 클래스로 처리함 — Motion initial은 SSR에 opacity 0으로 실려 하이드레이션 전까지 폼이 보이지 않음
- 토큰 만료는 루트 layout의 SessionWatcher가 expiresAt 시각 타이머와 새로고침 시 GET /api/me 복원으로 감지하고, 이미 만료된 세션은 마운트 직후 만료 처리함 — 요청 없이 열어 둔 탭도 만료 시각에 이동함
- 세션 종료(만료·401·로그아웃, expireSession·logout)는 router 이동 대신 window.location.replace로 /login을 전체 새로 불러옴 — 이전 사용자의 TanStack Query 캐시와 저장하지 않은 입력을 함께 버리고, React 밖인 API 클라이언트도 같은 함수로 이동함
- 만료·401로 /login에 전체 이동할 때(expireSession)는 만료 안내 토스트를 sessionStorage 플래그로 로그인 화면에 넘김
- 만료 이동(expireSession)과 인증 가드(AppShell)는 로그인 주소를 getLoginUrl 하나로 만들어 현재 경로를 returnTo 쿼리로 붙이되, 현재 경로가 /이면 붙이지 않음 — /는 returnTo가 없을 때의 기본 복귀 경로임
- 다른 탭에서 세션이 바뀌면(로그인·로그아웃·만료, localStorage.clear 포함) localStorage storage 이벤트(watchOtherTabs)로 감지해 현재 탭을 새로고침하고 그 탭의 저장하지 않은 입력 손실을 감수함 — 공용 PC에서 화면은 이전 사용자, 요청 토큰은 새 사용자로 어긋나는 것을 막음
- 만료 처리(expireSession)는 넘겨받은 토큰이 현재 저장된 세션 토큰과 같을 때만 세션을 지우고, 겹친 호출은 첫 호출만 동작함 — 다른 탭의 옛 만료 타이머나 옛 토큰 요청의 늦은 401이 새로 로그인한 세션을 지우지 않게 함
- API 클라이언트(apiGet·apiPost)는 토큰을 붙인 요청의 응답 코드 UNAUTHORIZED에만 만료 처리를 하고(상태 코드가 아닌 응답 코드로 분기), 토큰 없이 받은 401은 만료로 보지 않으며 INVALID_CREDENTIALS는 로그인 화면이 처리함
- 로그인 화면은 마운트 때 getSession()을 한 번만 읽어 유효 세션이면 /로 이동함 — useSession() 구독으로 하면 로그인 성공 직후 저장 알림이 / 이동을 다시 일으켜 returnTo 복귀 경로를 덮어씀
- 세션 만료 판정은 isSessionValid 하나로 하며 getSession은 저장된 세션만 읽고 만료 여부를 판정하지 않음 — 로그인 화면과 인증 가드가 같은 판정을 쓰게 함
- 현재 사용자 쿼리(authQueries.me)는 staleTime을 무한으로 두어 창 포커스마다 다시 부르지 않으며 쿼리 키에 토큰을 넣지 않음 — 새로고침 시 복원용이며, 세션이 끝나면 문서를 새로 불러 이전 사용자 캐시가 남지 않음

## 결정

- 토큰 만료를 요청 시점 401로만 처리하는 안은 기각함 — 요청 없이 열어 둔 탭은 만료 시각에 이동하지 못함

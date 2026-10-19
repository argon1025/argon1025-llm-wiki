---
description: 화면 검증을 위해 next dev 개발 서버를 띄울 때
---

# 개발 서버 실행

## 규칙

- worktree에서 화면을 검증할 때 포트 3101을 기본 체크아웃 개발 서버가 쓰고 있으면 다른 포트(예: `next dev -p 3109`)로 띄우고 백엔드 `CORS_ORIGINS`에 그 origin을 더함
- 로그인·만료 브라우저 검증은 저장소가 분리된 `{이름}.localhost:{포트}` origin에서 하고 백엔드 `CORS_ORIGINS`에 그 origin을 더함 — 세션이 localStorage라 같은 origin(`localhost:{포트}`)의 다른 탭과 storage 이벤트·만료 타이머를 공유함

## 함정

- Next.js 16 개발 서버는 같은 프로젝트 디렉터리에서 두 번째 `next dev`를 거부함(`Another next dev server is already running`) — 화면 검증은 이미 떠 있는 서버를 쓰거나 그 프로세스를 정리한 뒤 띄움
- 저장소를 분리하려고 개발 서버를 `127.0.0.1:{포트}`로 열면 하이드레이션되지 않음 — Next.js 16 개발 서버가 `allowedDevOrigins` 기본값으로 `127.0.0.1`의 `/_next` 개발 리소스 요청을 막고 `*.localhost`는 허용함

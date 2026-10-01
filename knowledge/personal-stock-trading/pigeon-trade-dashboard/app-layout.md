---
description: 대시보드 공통 레이아웃이나 페이지 최상위 태그를 추가·수정할 때
---

# 앱 레이아웃

## 규칙

- 공통 화면 레이아웃(사이드바+헤더)은 라우트 그룹 `(dashboard)`의 레이아웃(`DashboardLayout`)에 두고 루트 레이아웃(`RootLayout`)에는 두지 않음 — 로그인 등 레이아웃 밖 화면을 추가할 여지를 남김
- 라우트 그룹 `(dashboard)` 아래 페이지는 최상위 태그로 `<main>`을 쓰지 않고 `section`·`div`를 씀 — 레이아웃의 `SidebarInset`이 이미 `<main>`으로 렌더되어 `<main>`이 중첩됨

---
description: 테마 색·표면·글꼴·아이콘 토큰을 추가·변경하거나 카테고리 색을 배정할 때
---

# 테마 토큰

## 규칙

- 테마 표면은 background(메뉴·로고 영역과 페이지 바탕, sidebar 토큰 동일) > panel(라운드 콘텐츠 영역, 프로젝트 전용 토큰 bg-panel) > card 3단계임
- 다크 표면 위 비텍스트 대비 3:1을 맞춰야 하면 다크 모드 primary(--primary)만 밝은 톤 #465aff로 바꿈
- 런타임에 글꼴을 바꾸려면 next/font가 html에 거는 --font-urbanist·--font-pretendard를 덮어씀 — font-sans 같은 @theme inline 유틸리티는 값을 인라인해 --font-sans를 덮어써도 반영되지 않음
- 표·시간표에서 숫자 자릿수 정렬이 필요해지면 Pretendard tabular-nums 유틸리티를 도입함 — Urbanist 숫자에는 tabular-nums가 동작하지 않음
- base 레이어가 활성 button과 role=button 요소에 cursor: pointer를 복원함 — Tailwind CSS v4 preflight가 버튼 커서를 default로 둠
- lucide 아이콘 선 굵기는 기본 2 대신 --icon-stroke 1.25를 .lucide에 덮어씀 — utilities 레이어의 stroke-width 유틸리티 클래스가 이보다 우선함
- 과목·탐구 축 카테고리 색은 프론트엔드가 ID 기준 순환(category-((id - 1) % 8 + 1), categoryOf)으로 배정함 — 백엔드에 색 필드가 없음
- 과목·탐구 축 항목별 색 지정이 필요해지면 백엔드에 색 필드를 추가함 — ID 기준 순환은 9번째 항목부터 색이 반복됨

## 함정

- Primary 브랜드색(--primary)은 라이트·다크 공통 #373efb(흰 글자 대비 6.2:1)이며 다크 표면 위 체크·포커스 링·링크 대비 2.57~3.07:1은 비텍스트 기준 3:1 미달이나 감수한 값임(의도된 동작)
- 숫자는 Urbanist 비례폭 그대로라 표·시간표에서 자릿수가 세로로 어긋날 수 있음(정상) — Urbanist(Google Fonts)에 tnum 기능이 없어 Tailwind tabular-nums가 동작하지 않음
- 런타임 주입 CSS·인라인 style에서는 유틸리티 클래스나 calc(var(--radius) * 2.2)처럼 원본 변수로 계산함 — Tailwind는 소스가 참조한 @theme inline 변수만 :root에 출력해 참조되지 않은 --radius-3xl 등의 var()는 무효임

## 결정

- background를 콘텐츠 회색으로 두는 안은 버림 — base-maia TabsTrigger 활성 상태가 bg-background라 카드 위 활성 탭이 구분되지 않음
- 카테고리 색을 순서값 기준으로 배정하는 안은 버림 — 순서 변경·삭제 때 색이 바뀜

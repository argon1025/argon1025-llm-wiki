---
description: shadcn 컴포넌트를 추가·사용하거나 init·테마·Pretendard 폰트를 다룰 때
---

# shadcn Base UI

## 규칙

- 외부 예제·블록을 가져올 때는 Base UI 버전으로 받음 — 이 레포 shadcn/ui는 Base UI 기반(`--base base`, 스타일 `base-nova`)이고 외부 예제·블록은 대개 Radix 기준임
- shadcn init은 `--base base --preset nova`로 실행함 — shadcn CLI 4.21.0 init은 `--preset base-nova`를 거부(`Invalid preset`)하며 이 조합이 `components.json`의 `style`을 `base-nova`로 기록함
- shadcn 컴포넌트를 링크로 렌더할 때는 Radix `asChild`가 아닌 render prop(`render={<Link />}`)을 씀 — 컴포넌트가 Base UI 기반임
- Button을 링크로 렌더할 때는 `render`와 함께 `nativeButton={false}`를 지정함
- Pretendard 첫 방문 로딩 체감이 문제되면 dynamic-subset CSS로 전환함 — `next/font/local`로 등록한 `pretendard` 패키지 가변 woff2는 2.1MB 전체를 첫 방문에 받음

## 함정

- 생성 컴포넌트가 `@/lib/utils`가 아닌 `cn` npm 패키지를 직접 import하는 것은 정상 — shadcn 4.21 생성 코드의 `cn`은 clsx·tailwind-merge 조합이 아닌 `cn` 패키지이며 두 경로는 같은 함수임
- `shadcn` 패키지를 CLI 전용으로 보고 `devDependencies`로 옮기거나 제거하면 안 됨 — `globals.css`의 `@import 'shadcn/tailwind.css'`가 런타임 `dependencies`로 요구함
- `shadcn init`을 다시 실행하거나 테마를 재생성하면 `globals.css`의 `@theme inline`에 `--font-sans` 자기 참조가 써져 `--font-sans: var(--font-pretendard)` 연결이 끊김 — Pretendard가 적용되지 않음
- `npx shadcn add`는 기존 `button.tsx` 덮어쓰기를 대화형으로 물어 `--yes`만으로는 멈춤 — 비대화 실행 시 `yes n |`로 거부 응답을 넣어야 기존 생성물이 보존됨
- `TabsList`를 `overflow-x-auto` 요소로 감싸면 탭 우측에 스크롤바가 생김 — `TabsTrigger`(`base-nova`)가 탭 아래 `after:bottom-[-5px]` 선 요소를 둠

## 결정

- Radix 대신 Base UI 선택 — Radix는 공식 블록·서드파티 자료가 많으나 shadcn 4.21 기본값이 Base UI임
- Pretendard 로딩은 dynamic-subset CSS 대신 `next/font/local` 선택 — preload·fallback 크기 보정을 줌

---
description: React Compiler 린트에 막히는 effect·ref·아이콘 코드를 쓸 때
---

# React Compiler 린트 함정

## 적용 대상

- edu-fit-frontend의 eslint-config-next 16, react-hooks 7 린트(React Compiler 규칙)

## 규칙

- useEffect 안의 상태 변경은 prop 콜백이나 훅이 돌려준 함수 호출로 함(StudentDialogBody의 404 effect는 onClose prop 호출) — 이 호출은 react-hooks/set-state-in-effect를 통과함
- fileIconOf는 .map 콜백 안에서만 호출함 — 컴포넌트 최상위에서 const Icon = fileIconOf(...); <Icon />로 그리면 react-hooks/static-components로 막히고 .map 콜백 안은 통과함
- fileIconOf 밖의 아이콘 컴포넌트는 모듈 수준 배열의 구조 분해로 그림 — 렌더 중 고른 아이콘 컴포넌트를 컴포넌트 최상위에서 <Icon />로 그리면 react-hooks/static-components로 막힘
- 탐구 요청 폼 요소는 id(FORM_ID) 조회로 찾음 — useTaskForm이 반환한 객체 속성의 ref를 ref={form.formRef}로 넘기면 react-hooks/refs로 막힘

## 함정

- useEffect 안에서 같은 컴포넌트 setState를 감싼 지역 함수를 부르거나 useEffectEvent를 거쳐 setState를 불러도 react-hooks/set-state-in-effect로 막힘

---
description: Aurora 같은 React Bits WebGL 효과나 로고 애니메이션을 추가·수정할 때
---

# 시각 효과

## 규칙

- React Bits WebGL 컴포넌트(Aurora·FloatingLines) 복사본은 effect 정리 단계의 WEBGL_lose_context 해제 호출을 뺌 — 해제 호출이 있으면 개발 모드 StrictMode 재실행 시 같은 canvas가 잃은 컨텍스트를 받아 흰 화면이 됨
- React Bits WebGL 컴포넌트 복사본은 해제 호출을 뺀 곳만 '프로젝트 수정' 주석으로 표시하고 나머지는 원본 그대로 둠
- 목업을 일괄 삭제하면 lottie-react도 package.json에서 함께 제거함 — 목업 전용 의존성임

## 결정

- 확정 로그인 화면의 로고 그리기 애니메이션(LogoDraw)은 lottie-react 대신 Motion motion.path의 pathLength·pathOffset으로 재현함 — lottie-react 진입점이 엔진 3종을 함께 실어 /login 청크가 커짐

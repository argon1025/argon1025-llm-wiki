---
description: 린트·포맷 도구나 그 스크립트를 바꿀 때
---

# 린트·포맷 도구 구성

## 린트 도구 선택

- 린트는 ESLint + airbnb 대신 oxlint(type-aware) 규칙 강화로 운영함 — 원조 eslint-config-airbnb는 flat config 미지원·유지보수 중단이고 NestJS 12 스캐폴드가 oxlint를 기본 채택함(`.oxlintrc.json`)
- 기각한 대안은 커뮤니티판 eslint-config-airbnb-extended와 typescript-eslint strictTypeChecked이며 ESLint 재도입 비용이 기각 사유임
- oxlint의 style 카테고리는 켜지 않음 — 코드 스타일은 Prettier가 전담

## 포맷 스크립트 함정

- test/ 폴더가 없는 동안 포맷 스크립트(`package.json#scripts.format`)의 --no-error-on-unmatched-pattern을 빼면 prettier가 test/**/*.ts 미매칭으로 exit 2를 반환하며, 린트 스크립트(lint)의 oxlint는 없는 test/ 경로를 오류 없이 무시함

## oxlint 규칙 예외와 함정

- 런타임 검증 없는 단언(as T)은 no-unsafe-type-assertion 위반이므로 사유 주석과 oxlint-disable-next-line을 함께 둠
- spec의 의존성 대체는 단언(as unknown as mock) 대신 실제 인스턴스에 스파이(vi.spyOn)를 걸고, 설정 서비스(ConfigService)는 생성자 인자로 값을 주입해 만듦
- 읽기전용 매개변수 규칙(prefer-readonly-parameter-types, warn)은 끄지 않고 메서드를 readonly로 취급(treatMethodsAsReadonly)하며 수정 불가능한 외부 클래스(@nestjs/config의 ConfigService)만 예외(allow) 처리함
- 새 인터페이스·인자 타입은 필드에 readonly, Record에는 Readonly<>, it.each 테이블은 as const로 선언해야 경고가 없음
- 함수 길이 제한 규칙(max-lines-per-function)은 스펙 파일(*.spec.ts, *.e2e-spec.ts)에서만 끔

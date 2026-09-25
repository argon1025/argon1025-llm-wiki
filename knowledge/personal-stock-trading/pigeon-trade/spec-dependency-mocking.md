---
description: spec에서 의존성을 mock으로 대체할 때
type: convention
---

# spec의 의존성 대체

## spec의 의존성 대체

- spec의 의존성 대체는 `as unknown as` mock 객체 대신 실제 인스턴스에 `vi.spyOn`을 걸고 `ConfigService`는 생성자 인자로 값을 넣어 만듦 — `no-unsafe-type-assertion` 오류 회피

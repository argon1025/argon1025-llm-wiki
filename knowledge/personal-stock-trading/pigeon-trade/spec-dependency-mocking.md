---
description: spec에서 의존성을 mock으로 대체할 때
type: convention
---

# spec의 의존성 대체

## spec의 의존성 대체

- spec의 의존성 대체는 `as unknown as` mock 객체 대신 실제 인스턴스에 `vi.spyOn`을 걸고 `ConfigService`는 생성자 인자로 값을 넣어 만듦 — `no-unsafe-type-assertion` 오류 회피
- 엔티티 매니저(EntityManager)에 의존하는 클래스는 `new MikroORM(createMikroOrmConfig(더미 값))`의 `orm.em`으로 실제 인스턴스를 만들고 `vi.spyOn`으로 대체함 — MikroORM 7의 해당 생성은 DB에 연결하지 않음

## e2e 시각 고정

- 실DB e2e spec에서 시각을 고정할 때는 `vi.useFakeTimers({ toFake: ['Date'] })`로 Date만 대체함 — 전체 타이머를 대체하면 MySQL 드라이버 타이머와 테스트의 setTimeout 지연이 멈춤

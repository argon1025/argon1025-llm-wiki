---
description: 클래스 파일을 구성할 때
type: convention
---

# 클래스 파일 구성

## 구성

- 한 파일에 클래스 하나만 둠
- 변환·판정 로직은 파일 수준 함수 대신 클래스의 private 메서드로 둠
- 클래스 접미사는 manager 대신 service를 씀
- 서비스 입력·출력 인터페이스(Option·Result)는 서비스 파일에 두지 않고 도메인 interface/ 폴더에 서비스당 1파일({name}-service.interface.ts)로 둠
- export하지 않는 파일 내부 전용 타입도 같은 폴더의 interface/ 하위 파일로 분리함

---
name: solid
version: "0.1.0"
description: "SOLID 원칙 (Robert Martin) — SRP/OCP/LSP/ISP/DIP. OO 설계의 고전 원칙. 과적용 경고 포함. Dan North의 CUPID 대안 언급."
---

# SOLID Principles

## 한 줄 요약

객체지향 설계의 5원칙. **변경에 강한 코드를 만드는 사고 도구**. 단, 원칙에 코드를 맞추려 하면 역효과. 원칙은 *고통을 줄이는 렌즈*이지 체크리스트가 아니다.

## 이론 기원

- **Robert C. Martin (Uncle Bob)** — 1990s~2000s 초 정리, 2003 *Agile Software Development, Principles, Patterns, and Practices*에서 SOLID 두문자 확정. SOLID라는 두문자어는 Michael Feathers가 제안.
- 개별 원칙 기원:
  - **SRP**: Parnas, D. "On the Criteria to Be Used in Decomposing Systems into Modules." *CACM* (1972). "Information hiding"과 "변경 이유 분리"의 원형.
  - **OCP**: Meyer, B. *Object-Oriented Software Construction* (1988). 원전은 상속 기반, Martin이 다형성(인터페이스) 기반으로 재해석.
  - **LSP**: Liskov, B. "Data Abstraction and Hierarchy." *OOPSLA* (1987) keynote 및 Liskov & Wing, "A Behavioral Notion of Subtyping." *TOPLAS* (1994).
  - **ISP, DIP**: Martin 자신이 1990s Xerox 및 Object Mentor 컨설팅 경험에서 정리.

## 5원칙

### S — Single Responsibility Principle (SRP)
**"클래스는 변경해야 할 이유가 하나여야 한다."**
- "하나의 일을 한다"가 아님. *하나의 이해관계자/변경 축*
- 예: Report 클래스가 "계산 로직"과 "출력 형식" 둘 다 갖으면 → 분리
- 실무 신호: 커밋 메시지가 "A 때문에 바뀜 + B 때문에 바뀜"

### O — Open/Closed Principle (OCP)
**"확장에 열려 있고, 수정에 닫혀 있어야 한다."**
- 신규 동작 추가 시 *기존 코드를 건드리지 않고* 가능해야
- 방법: 다형성, 전략 패턴, 플러그인 구조
- 실무: switch/if-else 분기가 자주 늘어나면 OCP 위반 신호

### L — Liskov Substitution Principle (LSP)
**"하위 타입은 상위 타입을 깨트리지 않고 대체 가능해야 한다."**
- 자식이 부모의 계약 어기면 안 됨
- 고전 예: Square extends Rectangle 실패 사례
- 실무: `if (x instanceof Special)` 분기 필요 = LSP 위반

### I — Interface Segregation Principle (ISP)
**"클라이언트는 필요 없는 인터페이스에 의존하면 안 된다."**
- Fat interface → 작은 역할별 interface로 분리
- 예: `Worker { work(), eat() }` → Robot엔 eat() 필요 없음 → 분리
- 실무: 인터페이스 구현하며 미사용 메서드에 no-op·throw 하면 위반

### D — Dependency Inversion Principle (DIP)
**"고수준 모듈이 저수준 모듈에 의존하지 말고, 둘 다 추상에 의존하라."**
- 구체 클래스가 아닌 인터페이스에 의존
- DI(Dependency Injection)와 밀접 (DIP는 원칙, DI는 기법)
- `hexagonal` Architecture의 이론적 기반

## 언제 쓰나

- OO 언어로 장기 유지보수 코드베이스 작성 시
- 팀 온보딩 시 공통 언어로
- 코드 리뷰의 *논의 도구*
- 리팩터링 방향성 판단
- 테스트 어려운 코드 정리

## SOLID가 *주* 프레임워크로 뽑히는 상황 (아키텍처 수준 아님)

큰 구조(경계·팀·배포)가 앞서면 SOLID는 뒤로 밀리는 게 맞다. 하지만 다음은 SOLID가 중심:

1. **레거시 함수·클래스 단위 리팩터** — 모듈·서비스 경계는 그대로 두고 내부만 정리
2. **God class 분해** — 2000줄짜리 `OrderService.java` → SRP로 쪼개기
3. **switch 분기가 계속 늘어나는 결제 라우팅** — OCP + 전략 패턴으로 교체
4. **내부 SDK·라이브러리 인터페이스 설계** — ISP·DIP가 API 품질 결정
5. **코드 리뷰 공통 언어 정립** — 신입 온보딩 + PR 리뷰 가이드
6. **상속 계층 재설계** — LSP 위반 잡기
7. **"테스트하려면 DB 띄워야 하는" 함수** — DIP + `hexagonal`로 전환

### 구체 시나리오

```
상황: 3년 된 주문 서비스, `OrderProcessor` 2400줄
  - if (paymentType == ...) 분기 22개
  - 재고·배송·쿠폰·적립·알림 다 섞여 있음
  - 테스트는 통합 테스트 2개뿐
신호 매핑:
  - "건드리면 깨짐" → SRP·OCP 축적 위반
  - "테스트 안 됨" → DIP 없음 + I/O 결합
접근:
  1. SRP: 결제 라우팅 / 재고 차감 / 쿠폰 적용 / 알림을 분리
  2. OCP: paymentType switch → PaymentStrategy 인터페이스 + 구현체
  3. DIP: DB·외부 API를 인터페이스 뒤로 (→ hexagonal 자연 귀결)
  4. 테스트: 분리된 각 책임에 단위 테스트
결과: 2400줄 → 평균 200줄 클래스 10개 + 인터페이스 + fake
```

## 실전 팁

### 렌즈로 쓰기
- "이 코드 왜 건드리기 무섭지?" → SRP·OCP 점검
- "왜 테스트가 이렇게 길지?" → DIP·ISP 점검
- "상속 써도 될까?" → LSP 통과하는가

### 우선순위
- **DIP + SRP** 먼저. 가장 실용적 효과 큼.
- OCP는 *변경 패턴이 보인 뒤* 적용. 미리 하면 과설계.
- LSP는 상속 쓸 때만 의식.
- ISP는 큰 인터페이스 생겼을 때 분리 트리거.

## 안티패턴 & 과적용

- **클래스를 극단적으로 작게** — SRP 과적용. 10줄짜리 수십 개 파일은 읽기 더 어려움
- **모든 것을 인터페이스로** — DIP 과적용. 구체 1개뿐인 인터페이스는 의미 없음
- **추상화를 위한 추상화** — "혹시 나중에 바뀔 수도"로 계층 추가
- **SOLID = 리팩터 정답** — 아님. 중복 허용이 더 나을 때 많음 (Rule of Three)
- **원칙 이름만 팔고 적용 모호** — 코드 리뷰에서 "SRP 위반"만 말하면 토론 안 됨. 구체적 변경 축 지목 필요

## Dan North의 CUPID 대안 (2022)

SOLID가 과하다고 느낀다면:
- **C**omposable
- **U**nix philosophy
- **P**redictable
- **I**diomatic
- **D**omain-based

CUPID는 "속성" 기반, SOLID는 "규칙" 기반. 서로 배타 아님.

## 한계

1. **OO 외 패러다임엔 일부만 적용** — 함수형·데이터 중심엔 다른 원칙 (불변성, 순수성)
2. **원칙 이름 ≠ 이해** — 말로만 외우면 오해
3. **'상속으로 해결' 유혹 유도** — LSP 때문에 상속 쓰는 실수
4. **비즈니스 문제가 아님** — 어떤 도메인 구조인지는 `ddd` 몫

## 이 프레임워크와 함께 쓰는 것들

- `hexagonal` — DIP의 자연스러운 귀결
- `ddd` — SRP의 도메인 버전 (Aggregate 경계)
- `solid`는 *코드 수준*, `hexagonal`·`ddd`는 *구조 수준*

## 이 프레임워크가 *틀렸을 때*

- 단기 스크립트·프로토타입 → 과적용
- 절차적·함수형 코드 → 다른 원칙
- 도메인 경계 문제 → `ddd`

## 추가 학습

- Martin, R. *Agile Software Development, Principles, Patterns, and Practices.* (원전)
- Martin, R. *Clean Architecture.* (확장판)
- North, D. "CUPID — for joyful coding" (블로그, 2022).
- Freeman·Pryce. *Growing Object-Oriented Software, Guided by Tests.* (실무 적용)

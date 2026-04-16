---
name: ddd
version: "0.1.0"
description: "Domain-Driven Design (Eric Evans) — 복잡 비즈니스 도메인을 언어·경계·모델로 풀어냄. Strategic(bounded context·context map·ubiquitous language) + Tactical(entity·value object·aggregate·domain event)."
---

# Domain-Driven Design

## 한 줄 요약

**복잡한 비즈니스의 정수는 도메인 모델에 있다.** 기술 구조가 아니라 *도메인 언어와 경계*로 코드를 조직하라. 핵심은 전략(Strategic), 전술(Tactical)은 표현 수단.

## 이론 기원

- **Eric Evans** — *Domain-Driven Design: Tackling Complexity in the Heart of Software* (Addison-Wesley, 2003). 원전이자 "Strategic vs Tactical" 구분의 출처.
- **Vaughn Vernon** — *Implementing Domain-Driven Design* (Addison-Wesley, 2013). Evans 책을 실무 코드 수준으로 풀어냄.
- **Alberto Brandolini** — *Introducing EventStorming* (Leanpub, 2015~, 미완·계속 업데이트). 도메인 이벤트 기반 모델링 워크샵 기법.
- **Vlad Khononov** — *Learning Domain-Driven Design* (O'Reilly, 2021). 현재 표준 입문.
- 2014~2018 마이크로서비스 운동(Newman·Fowler)과 결합 — bounded context = 서비스 경계 공식화.
- 뿌리: Booch (1994), Fowler *Analysis Patterns* (1996)의 도메인 모델링 계보.

## Strategic (전략적) — 가장 중요

### 1. Ubiquitous Language (유비쿼터스 언어)
- 도메인 전문가와 개발자가 *동일한 용어* 사용
- 코드·문서·대화 모두 동일
- 번역이 필요 = 도메인 이해 실패

### 2. Bounded Context (경계 컨텍스트)
- 한 모델이 *일관되게* 유효한 범위
- "Customer"가 *주문* 컨텍스트와 *지원* 컨텍스트에서 다를 수 있음
- 마이크로서비스 경계 설정의 이론적 근거

### 3. Context Map
- bounded context 간 관계 종류:
  - **Partnership** — 양방향 조율
  - **Customer-Supplier** — 한쪽이 요구사항 결정
  - **Conformist** — 의존하는 쪽이 맞춤
  - **Anti-Corruption Layer (ACL)** — 외부 모델 번역 보호층
  - **Shared Kernel** — 소수의 공유 코드
  - **Open Host Service + Published Language** — 외부 공개 프로토콜

### 4. Subdomain 분류
- **Core Domain** — 경쟁 우위 영역. 최고 인재·정성.
- **Supporting** — 필요하지만 차별점 아님.
- **Generic** — 해결된 문제. 패키지·SaaS로 대체.

투자 우선순위: Core > Supporting > Generic. *Generic에 내부 개발 낭비 금지*.

## Tactical (전술적)

### Entity
- 고유 식별자(ID)로 구분. 속성이 바뀌어도 동일 객체.

### Value Object
- 속성으로만 판단. 불변. 비교는 값으로.
- 예: Money, Address, DateRange.

### Aggregate
- 일관성 단위. Aggregate Root가 내부 접근 통제.
- 트랜잭션 경계 = Aggregate 경계.
- **한 트랜잭션에 한 Aggregate 수정 원칙** (Vernon 2013, Ch.10)
- **작게 유지** — 큰 Aggregate는 낙관적 락 충돌률 폭증. Vernon의 4가지 경험칙:
  1. 진짜 불변식(invariant)만 경계 안에
  2. 작은 Aggregate 선호
  3. 다른 Aggregate는 *ID로* 참조
  4. Aggregate 간 일관성은 *eventually consistent* (도메인 이벤트로)

### Repository
- Aggregate 단위로만 로드·저장.
- 도메인 언어 인터페이스. Adapter는 secondary port.

### Domain Event
- 도메인에서 *이미 일어난 일*.
- Bounded context 간 통신의 비동기 매개.

### Domain Service
- Entity·Value Object에 속하지 않는 도메인 로직.
- 예: PaymentRouter, PricingCalculator.

## 언제 쓰나

- 비즈니스 로직이 복잡하고 변동 잦음
- 도메인 전문가와 상시 소통
- 여러 팀이 동일 제품에서 일함 (서비스 분할 필요)
- 장수 제품 (5년+ 유지보수)

## 실전 적용

### 서비스 분할의 출발
1. Event Storming 워크샵 (Alberto Brandolini)으로 도메인 이벤트 식별
2. 이벤트 클러스터 → bounded context 후보
3. Context Map 그리기
4. Core Domain 선정 → 투자 집중
5. 서비스 경계 = bounded context (팀 구조와 정렬)

### Aggregate 설계 룰
- 작게 유지 (large aggregate = 동시성 병목)
- 참조는 ID로, 객체 그래프 X
- 일관성 요구가 다르면 분리
- Eventually consistent between aggregates

## 안티패턴

- **Anemic Domain Model** — Entity에 getter/setter만, 로직은 Service에 몰림. DDD 아님.
- **"DDD" = Tactical 패턴 모음** — Strategic 없이 Tactical만 가져오면 실패
- **Core 착각** — 모두가 자기 도메인이 Core라 생각
- **Bounded Context 남용** — 단순 도메인에 10개 context 쪼개기
- **Ubiquitous Language 부재** — "고객", "사용자", "회원" 혼용

## 한계

1. **비용 큼** — 도메인 전문가 참여·워크샵·모델링 시간
2. **단순 도메인엔 과잉** — CRUD·단순 중개엔 Rails 스타일이 빠름
3. **학습 난이도** — Evans 원서는 어렵고, 실무 연결은 Vernon 필요
4. **Tactical 유혹** — 팀이 책 앞부분(Strategic) 안 읽고 Aggregate만 따라 함

## 한국 맥락 흔한 실패

- **"고객 vs 사용자 vs 회원 vs 이용자"** — 4개 용어를 법무·마케팅·개발이 각자 다르게 씀. Ubiquitous Language 실패 대표 사례.
- **결제·주문·정산 경계 흐림** — "주문이 결제를 호출"만 보이고 bounded context 미분리. 토스·쿠팡·네이버페이도 초기엔 겪은 문제.
- **규제가 만드는 경계** — 전자금융업·마이데이터·신용정보법이 각자 다른 제약(데이터 보관·접근 로그·동의 관리). 이게 bounded context 후보.
- **"파트장·팀장이 DDD 책 읽고 왔다"** — Strategic 논의 없이 Tactical(Aggregate·Repository)만 도입 → Anemic Model + 보일러플레이트. 흔한 한국 대기업 전환 실패 패턴.
- **도메인 전문가 부재** — 기획자를 도메인 전문가로 오해. 실제 업무 담당자(CS·정산·세무)와 대화하지 않으면 Ubiquitous Language 허상.

## 이 프레임워크와 함께 쓰는 것들

- `hexagonal` — 도메인과 I/O 분리 구조
- `event-sourcing-cqrs` — 도메인 이벤트를 진짜 "이벤트"로
- `modular-monolith` — bounded context = 모듈
- `team-topologies` — bounded context = 팀 경계

## 이 프레임워크가 *틀렸을 때*

- CRUD 중심 단순 앱 → DDD 풀스택 말고 얇은 레이어드 + `solid`
- 도메인 불명확 MVP → `ddd` 풀은 PMF 이후
- 기술 부채 정리 → `strangler-fig` 먼저

## 추가 학습

- Evans, E. *Domain-Driven Design.* (고전, 어려움)
- Vernon, V. *Implementing DDD.* (실무 연결)
- Brandolini, A. *Introducing EventStorming.*
- 현대: Khononov, V. *Learning Domain-Driven Design.* (2021, 현재 표준 입문)

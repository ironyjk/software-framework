---
name: hexagonal
version: "0.1.0"
description: "Hexagonal Architecture (Ports & Adapters) — Alistair Cockburn. 도메인 로직을 I/O(DB·HTTP·큐·UI)에서 격리. 테스트 용이성·교체 가능성·기술 독립성."
---

# Hexagonal Architecture (Ports & Adapters)

## 한 줄 요약

**도메인 로직은 가운데. I/O는 바깥. 둘은 포트(인터페이스)로만 소통.** 기술 스택·프레임워크가 바뀌어도 도메인은 그대로.

## 이론 기원

- **Alistair Cockburn** — "Hexagonal Architecture" (2005, alistair.cockburn.us). 원 아이디어는 1990s Crystal 방법론 당시 정립.
- 유사 개념:
  - **Onion Architecture** — Jeffrey Palermo 블로그 시리즈 (2008). 의존성 방향 더 엄격.
  - **Clean Architecture** — Robert Martin *Clean Architecture* (2017). Onion + Use Case 계층 명시.
  - **Ports and Adapters** — Cockburn이 2008년 이후 선호한 용어 (Hexagonal은 비유일 뿐).
- 셋 모두 핵심 아이디어 동일: *도메인을 가운데, 의존 방향 안쪽으로만*.
- 목적: "테스트하려고 DB 띄우기" 방지, 프레임워크 교체 가능성 확보.

## 구조

```
           ┌─────────────────────┐
   Driving │  Primary Adapters   │   (HTTP, CLI, gRPC, Scheduler)
   side    │  (inbound)          │
           └─────────┬───────────┘
                     │ Primary Port
           ┌─────────▼───────────┐
           │    Domain Core      │   ← 여기만 비즈니스 로직
           │  (entities·rules)   │
           └─────────┬───────────┘
                     │ Secondary Port
           ┌─────────▼───────────┐
   Driven  │  Secondary Adapters │   (DB, External API, MQ, Cache)
   side    │  (outbound)         │
           └─────────────────────┘
```

### 용어

- **Port** — 도메인이 외부와 상호작용하는 인터페이스. 도메인 언어로 정의.
- **Adapter** — 포트를 특정 기술(PostgreSQL, Kafka, REST)로 구현.
- **Driving (Primary)** — 바깥에서 도메인을 *호출*하는 쪽 (HTTP 컨트롤러, CLI).
- **Driven (Secondary)** — 도메인이 *바깥에* 명령하는 쪽 (Repository, EventPublisher).

## 핵심 규칙

1. **의존 방향 단방향**: Adapter → Port → Domain. *Domain은 Adapter를 모른다*.
2. **Port는 도메인이 소유**: DB 용어(row, table) 아닌 도메인 용어(Order, Customer).
3. **프레임워크 침투 방지**: Spring·Django 어노테이션·베이스클래스가 Domain에 스며들면 실패.
4. **Adapter는 교체 가능**: PostgreSQL→MongoDB는 Adapter 교체만.

## 언제 쓰나

- 도메인 로직 테스트를 *DB·네트워크 없이* 빠르게 돌리고 싶을 때
- 외부 시스템(PG사, 은행 API)이 유동적일 때 — 교체 가능성
- 장기 유지보수 제품
- 여러 진입점 (HTTP + CLI + 배치)

## 실전 적용

### 포트 설계 예시 (결제 도메인)
```
// Primary Ports (도메인이 "제공"하는 유스케이스)
interface ChargePayment {
  execute(cmd: ChargeCommand): ChargeResult
}

// Secondary Ports (도메인이 "필요로 하는" 외부)
interface PaymentGateway {
  charge(req: ChargeRequest): GatewayResponse
}
interface OrderRepository {
  findById(id: OrderId): Order | null
  save(o: Order): void
}
```

### 테스트 전략
- 단위: Domain + Fake Adapters (메모리 구현)
- 통합: 실제 Adapters에 contract test
- E2E: 한두 개 행복 경로만

### 점진 도입
신규 레포: 첫날부터.
기존 레포: 가장 변동 잦은 도메인 1개를 Adapter로 래핑, 점차 확대.

## 안티패턴

- **Port에 Response DTO 노출** — Port에 Spring ResponseEntity가 보이면 실패
- **Adapter에 도메인 규칙** — "DB 트리거에 비즈니스 로직" 전형적 실패
- **Domain이 ORM 엔티티** — JPA @Entity가 Domain 클래스이면 강결합
- **과적용**: CRUD 90%인 단순 API에도 포트·어댑터 풀스택 → 오버엔지니어링

## 한계

1. **보일러플레이트 증가** — 포트·어댑터·매퍼 레이어. 작은 앱엔 과잉.
2. **매퍼 비용** — Adapter ↔ Domain 변환 코드 지속 관리
3. **팀 학습 곡선** — 경계를 지킬 규율 필요. 약한 팀은 오히려 혼란.
4. **유행에 유혹된 도입** — "클린 아키텍처 쓰자"로 출발하면 실패 빈번. 고통이 있어야 정당화됨.

## 언제 *쓰지 마라*

- MVP 단계의 짧은 피드백 루프
- CRUD 중심의 얇은 서비스
- 팀이 3명 미만이고 제품 수명 6개월 미만 예상

## 이 프레임워크가 *보완 필요할 때*

- 도메인 복잡도 높음 → `ddd` (Hexagonal은 경계, DDD는 모델)
- 금융 원장·상태 복원 필요 → `event-sourcing-cqrs`
- 코드 수준 의존성 관리 → `solid`

## 추가 학습

- Cockburn, A. "Hexagonal Architecture." (원문, 2005)
- Martin, R. *Clean Architecture.* (유사 개념)
- Vernon, V. *Implementing Domain-Driven Design.* Ch. 4.

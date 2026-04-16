---
name: event-sourcing-cqrs
version: "0.1.0"
description: "Event Sourcing + CQRS (Greg Young) — 상태 대신 이벤트를 저장, 상태는 이벤트의 fold. 쓰기·읽기 모델 분리. 금융 원장·감사·시간 여행·읽기 부하 분리에 강력."
---

# Event Sourcing + CQRS

## 한 줄 요약

**State = Fold(Events)**. 현재 상태를 저장하지 말고 *상태를 바꾼 이벤트들*을 저장. 읽기는 별도의 최적화된 모델로 투영(projection).

## 이론 기원

- **Greg Young** — CQRS 개념 정립 (2010s). ES는 더 오랜 뿌리 (회계 원장·SCM).
- **Martin Fowler** — Event Sourcing 정리.
- **Udi Dahan** — SOA·메시지 기반 설계와 결합.
- 뿌리: 회계의 복식부기. "잔액"이 아닌 "거래" 기록.

## 두 가지 개념 (독립적 사용 가능)

### Event Sourcing (ES)
- 쓰기 = 이벤트 append
- 상태 = 이벤트들의 fold
- Event Store = 변경 불가 append-only 로그
- 장점: 완전한 감사 추적, 시간 여행(rebuild to any point), 도메인 이벤트 자연스러움
- 단점: 스키마 진화 복잡, 쿼리 어려움

### CQRS (Command Query Responsibility Segregation)
- Command (쓰기) 모델 vs Query (읽기) 모델 분리
- 각자 다른 DB·스키마·일관성 모델 가능
- 장점: 읽기 최적화, 스케일 독립, 팀 분리
- 단점: eventual consistency, 동기화 복잡성

### 둘 합치면
- ES가 source of truth
- CQRS read model이 이벤트 구독해 최신화
- 매우 강력·매우 복잡

## 언제 쓰나

- **감사·규제 필수 도메인** — 금융 원장, 의료 기록, 법적 거래
- **상태 히스토리 자체가 가치** — "언제 무엇이 바뀌었나"
- **읽기 부하 >> 쓰기 부하** — 별도 read model
- **복잡한 도메인 이벤트** — 비즈니스가 이벤트 언어로 자연
- **시간 여행 필요** — 과거 시점 시뮬레이션, what-if 분석

### 한국 금융 맥락에서의 트리거

- **전자금융업자 감독 보고** — 금감원·금융위 전자금융사고 보고 시 거래 흐름 재구성 필요 → ES가 자연스러움
- **PG·카드사 정산 주기** — T+2·T+3 정산에서 시점별 잔액 재구성이 상시 필요
- **마이데이터 API** — 본인신용정보 제공 이력 감사 추적 법정 요구
- **가상자산 사업자 (VASP)** — 특정금융정보법 거래내역 보관·제출
- **보험·증권 영업 로그** — 자본시장법·보험업법 기록 보존 의무
- **토스·카카오페이·네이버페이 사례** — 사용자 잔액·거래 조회는 projection, 원장은 append-only

## 실전 적용

### 토스·카카오페이 같은 결제 시스템
```
Command: ChargeRequested
Event: PaymentInitiated
      → PaymentAuthorized (by bank)
      → PaymentCaptured
      → PaymentSettled

Read models (각자 투영):
  - user-transaction-history (개인 뷰)
  - merchant-daily-settlement (가맹점 정산)
  - compliance-audit-log (규제 보고)
  - fraud-ml-features (ML 파이프라인)
```

### Event Store 선택
- 전용: EventStoreDB, Marten
- 범용 메시지: Kafka + KSQL (엄밀한 ES엔 한계)
- RDB 위 구현: outbox 패턴

### Projection (Read Model) 생성
- 이벤트 스트림 구독
- 최적화된 스키마로 유지
- *재생 가능* — 스키마 바꾸면 처음부터 재구축
- Eventually consistent (몇 ms~s 시차)

## 스키마 진화

- 이벤트는 *이미 일어난 사실* → 수정 불가
- 신 필드 추가 시: versioning (`OrderPlacedV1`, `V2`)
- Upcasting: 오래된 이벤트를 읽을 때 최신 형태로 변환
- *이벤트 재명명·삭제 금지* — 호환 레이어로 관리

## Concurrency·일관성 모델

- **Optimistic concurrency** — Aggregate 단위로 기대 버전(expected version)으로 append. 충돌 시 재시도 또는 command 리젝트. EventStoreDB·Marten 기본.
- **Stream per Aggregate** — 한 Aggregate의 이벤트는 단일 스트림 (e.g. `order-12345`). 동일 스트림 내 순서 보장.
- **Causal consistency** (스트림 내) vs **Eventual consistency** (스트림 간·projection 간)
- **Exactly-once processing** — Kafka 기반 ES에서 특히 어려움. Idempotent consumer + dedup key 설계 필요.
- **Snapshot 정책** — Aggregate 재구성 비용 < 쿼리 허용 지연일 때만. 보통 N개 이벤트마다 (N=50~1000) 또는 시간 기반.
- **Projection 복원** — 스키마 변경 시 전체 이벤트 리플레이. 대규모에선 수 시간~일 단위. 병렬 projector + partition 필요.
- **CAP 관점** — ES는 AP 성향 (write는 available, read projection은 eventually consistent).

## 안티패턴

- **Event Sourcing 없이 CQRS만** — 흔함. 단순 read model 분리는 "CQRS-lite"로 괜찮음
- **"모든 것을 Event Sourcing"** — 대부분의 CRUD엔 과잉
- **이벤트가 CRUD 이름**: `UserUpdated` 같은 무의미 이벤트 → 도메인 사건 아닌 DB 변경 log
- **Aggregate가 너무 큼** — 이벤트 스트림 리플레이 지연 폭발
- **Query 모델이 쓰기도 함** — CQRS 위반, 일관성 지옥
- **Snapshot 없이 긴 히스토리** — Aggregate 재구성 비용 폭증

## 한계

1. **학습 곡선 급함** — 팀 전체 mental model 전환 필요
2. **운영 복잡도** — projection lag 모니터링, replay 재해복구
3. **스키마 진화 정치** — 이벤트 계약이 team 간 경계
4. **Eventually consistent UX** — "방금 결제했는데 잔액 안 바뀜" 사용자 대응
5. **작은 팀·단순 도메인엔 과잉**

## 이 프레임워크와 함께 쓰는 것들

- `ddd` — 도메인 이벤트가 ES 이벤트. 함께 쓰면 자연스러움.
- `resilience-patterns` — 비동기·eventual consistency의 부작용 관리
- `observability` — projection lag·이벤트 쌓임 감시

## 이 프레임워크가 *틀렸을 때*

- 단순 CRUD → 과적용
- 강한 즉시 일관성 필수 (의료 투약 기록 등) → ES 가능하지만 동기 projection 필요
- 저부하·초기 MVP → ES의 감사 가치 < 복잡도 비용

## 추가 학습

- Young, G. "CQRS Documents" (PDF, 무료).
- Dahan, U. 블로그 및 NServiceBus 자료.
- Vernon, V. *Implementing Domain-Driven Design.* Ch. 8.
- EventStoreDB 문서 (eventstore.com).

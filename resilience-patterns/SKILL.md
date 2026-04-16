---
name: resilience-patterns
version: "0.1.0"
description: "Resilience Patterns — Circuit Breaker·Bulkhead·Back-pressure·Rate Limiting·Retry with jitter·Timeout. Nygard *Release It!* 기반. 외부 의존성 장애가 내 서비스를 끌어내리지 못하게."
---

# Resilience Patterns

## 한 줄 요약

**네 서비스는 네 것만 망가트릴 수 있게.** 외부 의존성(은행 API·MQ·DB)의 장애·느림이 전파되지 않도록 차단·격리·제어하는 패턴 묶음.

## 이론 기원

- **Michael Nygard** — *Release It!* (2007, 2nd ed 2018). 현장의 장애 사례에서 도출된 패턴.
- **Netflix Hystrix** (2012) — Circuit Breaker 대중화. 2018 유지보수 종료 후 Resilience4j가 계승.
- **Reactive Manifesto / Reactive Streams** — back-pressure 표준화.

## 핵심 패턴

### 1. Timeout
**모든 외부 호출에 시한을 걸어라.**
- 무한 대기는 리소스 고갈의 근원
- Connect / read / total 3단계 구분
- 권장: SLO에서 역산한 값. p99 + buffer.

### 2. Retry + Backoff + Jitter
**재시도는 똑똑하게.**
- 즉시 재시도 = 트래픽 폭증 (thundering herd)
- Exponential backoff: 1s, 2s, 4s, 8s...
- **Jitter 필수**: 모든 클라이언트가 동기화된 재시도 방지
- *멱등성* 없는 연산엔 조심 (결제 중복 방지)

### 3. Circuit Breaker (회로 차단기)
**연속 실패 시 자동으로 호출 차단.**
- 상태: CLOSED(정상) → OPEN(차단) → HALF-OPEN(시험) → CLOSED
- 실패율·응답시간 임계치 기반
- 차단 동안 fallback 또는 fast-fail
- 외부 API 복구 중 우리 서비스가 압박 안 주기

### 4. Bulkhead (격벽)
**리소스를 분리해서 한쪽 터져도 다른 쪽은 생존.**
- 스레드 풀 분리: API A용 풀 ≠ API B용 풀
- 커넥션 풀 분리
- 배 격벽 비유 — 물 찬 칸만 침수, 전체 안 가라앉음

### 5. Back-pressure
**받는 쪽이 "천천히 보내라" 신호할 수 있어야.**
- Reactive Streams, Kafka consumer lag 기반 제어
- 큐잉 무한 증가 방지
- 생산자가 속도 조절

### 6. Rate Limiting / Throttling
**초당 호출 수 상한.**
- Token bucket, Leaky bucket, Fixed window, Sliding window
- 내부 보호 + 외부 호출 쿼터 관리
- API Gateway·Redis·Envoy에 주로 위치

### 7. Load Shedding
**과부하 시 일부 요청을 포기.**
- 큐 가득 차면 즉시 reject
- 우선순위 기반 셰딩 (중요 트래픽만 통과)
- 503 + Retry-After

### 8. Dead Letter Queue (DLQ)
**처리 실패 메시지 격리.**
- 무한 재시도 대신 DLQ로 이동
- 별도 프로세스·수동 조사
- 원인 분석 후 복원

### 9. Idempotency
**같은 요청을 여러 번 받아도 같은 결과.**
- Idempotency-Key 헤더 관례
- 재시도 안전성의 기반
- 결제·주문 도메인 필수

### 10. Graceful Degradation
**핵심 기능 유지하며 부가 기능 포기.**
- 추천이 죽어도 결제는 돌게
- Fallback 응답·캐시 값
- 사용자에게 축소된 기능 노출

## 언제 쓰나

- 외부 API·DB·MQ 의존 있는 모든 서비스
- 금융·결제·예약 등 장애 비용 큰 도메인
- 초당 수천 요청 이상 규모
- SLA·SLO 준수 의무

## 실전 적용

### 레이어별 기본 세트
- **외부 API 호출**: timeout + retry(backoff+jitter) + circuit breaker + idempotency
- **내부 서비스 간**: timeout + bulkhead + rate limit
- **MQ consumer**: back-pressure + DLQ
- **DB**: connection pool + timeout + (필요 시) circuit breaker
- **API Gateway**: rate limit + load shedding

### 라이브러리
- Java/Kotlin: Resilience4j, Spring Cloud Circuit Breaker (토스·카카오뱅크 메인스트림)
- Node: opossum (CB), bottleneck (RL)
- Go: sony/gobreaker, uber-go/ratelimit
- Python: pybreaker, tenacity (retry)
- 서비스 메시(Istio·Linkerd): 언어 독립

### 한국 금융·결제 현장 패턴

- **카드사 12곳 연동**: 각 카드사별로 bulkhead(전용 스레드풀·커넥션풀). 한 카드사 지연이 타 카드사 호출 풀 소진하지 않도록 필수.
- **PG 다중화 (NICE·KCP·이니시스 등)**: primary PG 장애 시 secondary로 fallback. Circuit breaker + fallback routing.
- **은행 실시간 이체 (금결원 펌뱅킹·오픈뱅킹)**: 초당 호출 쿼터 + 시간대별 은행 점검 시간(보통 23:30~00:30) 대응. Retry는 idempotency-key 필수 (중복 송금 = 사고).
- **KFTC·금결원 장애**: 공통 인프라 장애는 CB 무력. Degradation(해당 이체 기능만 차단, 타 기능 유지) 공지.
- **FDS(Fraud Detection System) 호출**: 동기 호출이면 FDS 지연이 결제 p99 망침. 비동기 + 타임아웃 + fail-open/closed 정책 결정 필요.
- **금감원·PCI-DSS 대응**: load shedding·rate limit 로그는 보관 요구 대상.

### 테스트 (Chaos)
- Chaos Monkey 스타일: 프로덕션·스테이징에서 의도적 실패 주입
- toxiproxy, Gremlin, Litmus
- 패턴 작동 검증 없으면 "장식"에 불과

## 안티패턴

- **Retry without backoff** — 장애 증폭기
- **Retry without idempotency check** — 중복 결제
- **Circuit Breaker 없이 무한 타임아웃** — 리소스 고갈
- **모든 호출을 한 스레드풀** — 한 서비스 장애가 전체 마비
- **Rate limit만, observability 없음** — 누가 왜 제한됐는지 모름
- **패턴 라이브러리 깔고 기본 설정** — 도메인별 튜닝 없으면 효과 없음

## 한계

1. **튜닝 어렵다** — 임계치·타임아웃 숫자는 경험·실험으로만
2. **감추는 효과도 있음** — Circuit breaker가 근본원인 가시성 감소 (→ `observability` 필수)
3. **과적용 시 복잡도** — 작은 서비스에 10개 패턴 = 운영 부담
4. **테스트 없이는 믿을 수 없음** — Chaos testing 동반 필요
5. **Upstream 장애 근본 해결 아님** — 시간 벌기

## 이 프레임워크와 함께 쓰는 것들

- `observability` — 패턴 작동·임계치 조정의 근거 데이터
- `event-sourcing-cqrs` — 비동기·eventual consistency 결합 시 back-pressure 필수
- `twelve-factor` — disposability·concurrency가 전제

## 이 프레임워크가 *틀렸을 때*

- 단일 프로세스·외부 의존 없는 앱
- 실험 단계 프로토타입
- 근본원인 추적 필요 → `scientific-debugging`

## 추가 학습

- Nygard, M. *Release It!* (필독)
- Google SRE. *Site Reliability Workbook.* Ch. "Managing Load"
- Netflix Tech Blog — Hystrix·Concurrency Limits 글
- AWS Well-Architected — Reliability Pillar

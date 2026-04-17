---
name: resilience-patterns
version: "0.1.0"
description: "Resilience Patterns — Circuit Breaker·Bulkhead·Back-pressure·Rate Limiting·Retry with jitter·Timeout. Based on Nygard's *Release It!* Keep external dependency failures from dragging your service down."
---

# Resilience Patterns

## One-Line Summary

**Your service should only be able to break itself.** A bundle of patterns that block, isolate, and control so that failures and slowdowns of external dependencies (bank APIs, MQs, DBs) don't propagate.

## Theoretical Origin

- **Michael Nygard** — *Release It!* (2007, 2nd ed 2018). Patterns derived from real-world failure cases.
- **Netflix Hystrix** (2012) — popularized the Circuit Breaker. After maintenance ended in 2018, Resilience4j took over.
- **Reactive Manifesto / Reactive Streams** — standardized back-pressure.

## Core Patterns

### 1. Timeout
**Put a deadline on every external call.**
- Infinite waiting is the root of resource exhaustion
- Separate into 3 stages: connect / read / total
- Recommended: a value derived backward from SLO. p99 + buffer.

### 2. Retry + Backoff + Jitter
**Retry smartly.**
- Immediate retry = traffic surge (thundering herd)
- Exponential backoff: 1s, 2s, 4s, 8s...
- **Jitter is essential**: prevents all clients from synchronized retries
- Be careful with non-*idempotent* operations (prevent duplicate payments)

### 3. Circuit Breaker
**Automatically cut off calls on consecutive failures.**
- States: CLOSED (normal) → OPEN (cut off) → HALF-OPEN (trial) → CLOSED
- Based on failure rate and response time thresholds
- Fallback or fast-fail while cut off
- Don't pressure our service while the external API is recovering

### 4. Bulkhead
**Separate resources so that if one side blows up, the other survives.**
- Thread pool separation: pool for API A ≠ pool for API B
- Connection pool separation
- Analogy to ship bulkheads — only flooded compartments are submerged, the whole ship doesn't sink

### 5. Back-pressure
**The receiving side must be able to signal "slow down."**
- Reactive Streams, control based on Kafka consumer lag
- Prevents unbounded queue growth
- Producer adjusts speed

### 6. Rate Limiting / Throttling
**Upper limit on calls per second.**
- Token bucket, Leaky bucket, Fixed window, Sliding window
- Internal protection + external call quota management
- Typically located at API Gateway, Redis, Envoy

### 7. Load Shedding
**Abandon some requests when overloaded.**
- Reject immediately when the queue is full
- Priority-based shedding (only critical traffic passes)
- 503 + Retry-After

### 8. Dead Letter Queue (DLQ)
**Isolate messages that failed processing.**
- Move to DLQ instead of infinite retries
- Separate process, manual investigation
- Restore after root cause analysis

### 9. Idempotency
**Same result even if the same request is received multiple times.**
- Idempotency-Key header convention
- Foundation of retry safety
- Essential in payment and order domains

### 10. Graceful Degradation
**Maintain core functions while giving up auxiliary ones.**
- Even if recommendations die, payments keep running
- Fallback responses, cached values
- Expose reduced functionality to users

## When to Use

- Any service with external API, DB, or MQ dependencies
- Domains with high failure costs such as finance, payments, and reservations
- Scale of thousands of requests per second or more
- SLA/SLO compliance obligations

## Practical Application

### Default Sets per Layer
- **External API calls**: timeout + retry (backoff+jitter) + circuit breaker + idempotency
- **Inter-internal-service**: timeout + bulkhead + rate limit
- **MQ consumer**: back-pressure + DLQ
- **DB**: connection pool + timeout + (if needed) circuit breaker
- **API Gateway**: rate limit + load shedding

### Libraries
- Java/Kotlin: Resilience4j, Spring Cloud Circuit Breaker (mainstream at Toss, Kakao Bank)
- Node: opossum (CB), bottleneck (RL)
- Go: sony/gobreaker, uber-go/ratelimit
- Python: pybreaker, tenacity (retry)
- Service mesh (Istio, Linkerd): language-independent

### Korean Finance and Payment Field Patterns

- **Integration with 12 card companies**: bulkhead per card company (dedicated thread pool, connection pool). Essential so that a delay at one card company doesn't exhaust the call pool for other card companies.
- **PG multiplexing (NICE, KCP, Inicis, etc.)**: fallback to secondary when the primary PG fails. Circuit breaker + fallback routing.
- **Bank real-time transfers (KFTC firm banking, open banking)**: per-second call quota + handling of bank maintenance windows by time of day (usually 23:30–00:30). Idempotency-key is essential for retries (duplicate transfer = incident).
- **KFTC outages**: CB is powerless against common infrastructure outages. Degradation (cut off only the affected transfer function, keep other functions) notices.
- **FDS (Fraud Detection System) calls**: If synchronous, FDS delays wreck payment p99. Need to decide on async + timeout + fail-open/closed policy.
- **FSS (Financial Supervisory Service)·PCI-DSS compliance**: load shedding and rate limit logs are subject to retention requirements.

### Testing (Chaos)
- Chaos Monkey style: intentional failure injection in production and staging
- toxiproxy, Gremlin, Litmus
- Without verifying pattern operation, it's just "decoration"

## Antipatterns

- **Retry without backoff** — failure amplifier
- **Retry without idempotency check** — duplicate payments
- **Infinite timeout without Circuit Breaker** — resource exhaustion
- **All calls in a single thread pool** — failure of one service paralyzes the whole
- **Rate limit only, no observability** — don't know who was limited or why
- **Installing pattern libraries with default settings** — no effect without per-domain tuning

## Limitations

1. **Tuning is hard** — threshold and timeout numbers only from experience and experimentation
2. **Can also be a concealer** — Circuit breakers reduce root cause visibility (→ `observability` essential)
3. **Complexity when over-applied** — 10 patterns on a small service = operational burden
4. **Can't be trusted without testing** — Chaos testing must accompany
5. **Not a fundamental fix for upstream outages** — buying time

## What to Use Alongside This Framework

- `observability` — data backing pattern operation and threshold adjustments
- `event-sourcing-cqrs` — back-pressure essential when combining async and eventual consistency
- `twelve-factor` — disposability and concurrency are prerequisites

## When This Framework Is *Wrong*

- Single-process apps with no external dependencies
- Experimental-stage prototypes
- When root cause tracing is needed → `scientific-debugging`

## Further Reading

- Nygard, M. *Release It!* (must-read)
- Google SRE. *Site Reliability Workbook.* Ch. "Managing Load"
- Netflix Tech Blog — articles on Hystrix and Concurrency Limits
- AWS Well-Architected — Reliability Pillar

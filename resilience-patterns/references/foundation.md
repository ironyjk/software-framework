# Resilience Patterns -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

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


## Further Reading

- Nygard, M. *Release It!* (must-read)
- Google SRE. *Site Reliability Workbook.* Ch. "Managing Load"
- Netflix Tech Blog — articles on Hystrix and Concurrency Limits
- AWS Well-Architected — Reliability Pillar
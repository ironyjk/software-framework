---
name: resilience-patterns
version: "0.2.0"
description: "Resilience Patterns — Circuit Breaker·Bulkhead·Back-pressure·Rate Limiting·Retry with jitter·Timeout. Based on Nygard's *Release It!* Keep external dependency failures from dragging your service down."
---

# Resilience Patterns

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

**Your service should only be able to break itself.** A bundle of patterns that block, isolate, and control so that failures and slowdowns of external dependencies (bank APIs, MQs, DBs) don't propagate.


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

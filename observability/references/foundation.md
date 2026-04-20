# Observability -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

## Theoretical Origins

- **Brendan Gregg** — USE method (2012). Resource-centric.
- **Tom Wilkie** — RED method (2015). Request-centric.
- **Google SRE** — Four Golden Signals (*Site Reliability Engineering*, 2016).
- **Charity Majors · Honeycomb** — Popularized "high-cardinality observability."


## Three Standard Lenses

### 1. USE (Resource-centric, Gregg)
*For every resource*:
- **U**tilization — Percentage of time in use
- **S**aturation — Queue depth / degree of saturation
- **E**rrors — Error count

Applies to: CPU, memory, disk, network, file descriptors, thread / connection pools.

### 2. RED (Service-centric, Wilkie)
*For every requestable service*:
- **R**ate — Requests per second
- **E**rrors — Failure rate
- **D**uration — Response time (p50·p95·p99)

The standard for microservices environments. Complements USE with the "user experience" dimension.

### 3. Four Golden Signals (Google SRE)
- **Latency** — Separated for success and failure
- **Traffic** — Load on the system
- **Errors** — Failure rate
- **Saturation** — Saturation of the most constrained resource

An integrated version of USE + RED. Apply to user-facing systems first.


## Three Pillars of Telemetry

| Type | Purpose | Cost |
|---|---|---|
| **Metrics** | Dashboards · alerts · trends | Low |
| **Logs** | Event detail · investigation | Medium |
| **Traces** | Distributed request paths · bottleneck identification | Medium–High |

Modern practice: bundle **structured logs + exemplars + traces** under the OTel (OpenTelemetry) standard.


## High Cardinality and Sampling

- Cardinality = number of unique tag combinations (explodes when including user_id, request_id)
- Traditional metrics storage (Prometheus, etc.) has cardinality limits
- High-cardinality observability (Honeycomb · Grafana Loki) → enables tail-latency tracking for specific users / requests
- Sampling: head-based vs tail-based. Prioritize preserving errors and slow requests.


## Further Learning

- Gregg, B. *Systems Performance.* (USE method)
- Google SRE. *Site Reliability Engineering*, *SRE Workbook* (free online, sre.google).
- Majors, C., Fong-Jones, L., Miranda, G. *Observability Engineering.* O'Reilly, 2022.
- OpenTelemetry (opentelemetry.io)
- Pinpoint (github.com/pinpoint-apm/pinpoint) — Naver open source
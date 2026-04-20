---
name: observability
version: "0.2.0"
description: "Observability — Infer internal system state from external signals using USE (Gregg) + RED (Wilkie) + Four Golden Signals (Google SRE). 3 pillars (metrics/logs/traces). The standard lens for performance and incident diagnosis."
---

# Observability

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-line Summary

Can you ask about *internal state* through *external signals*? The extent to which you can is your observability. Every time you ask a performance, incident, or capacity question, look through the same framework.


## Key Metric Examples

| Metric | Good | Warning |
|---|---|---|
| API p99 latency | < service SLO | Approaching SLO |
| Error rate | < 0.1% | 1%+ |
| CPU util | < 70% | > 85% |
| DB conn pool sat | < 50% | > 80% |
| GC pause | < 100ms | > 500ms |


## When to Use

- Starting point for performance degradation analysis
- Capacity planning (current saturation → preparing for growth)
- SLO definition · compliance monitoring
- First lens for incident diagnosis
- Observing deployment impact (canary · blue-green)


## Practical Application

### Incident Diagnosis Order
1. **Golden Signals dashboard** — Where's red?
2. Identify affected services with **RED** — which of rate · error · duration?
3. Identify resource bottleneck with **USE** — CPU · memory · pool · disk?
4. Find latency in specific segments of the request path using **Traces**
5. Confirm root-cause stacks and error messages in **Logs**

### SLI/SLO Design
- SLI = measurable indicator (p99 latency, error rate)
- SLO = target (last 30 days p99 < 300ms, 99.9%)
- Error budget = experimentation possible within (1 - SLO)
- Alerting based on SLO burn rate (Google SRE Workbook)


## Antipatterns

- **"Store everything"** — cost explosion, signal-to-noise drops
- **Alert fatigue** — spraying threshold alerts → real signals missed
- **Looking only at CPU%** — misjudgment from a single metric without saturation · errors
- **Judging latency by average** — p50 lies. p95/p99 are essential
- **Cardinality explosion** — tagging with user_id (→ index explosion)


## Limitations

1. **Instrumentation cost** — Cannot collect, store, and query every signal. Prioritization required.
2. **Only sees known unknowns** — Can only see the signals already attached. Unknown unknowns require high-cardinality tools.
3. **Configuration lag** — Common to realize the needed signal is missing only after an incident strikes
4. **Correlation ≠ causation** — Observation tells you *where* but not *why* → `scientific-debugging`


## When This Framework Is *Wrong*

- Root cause analysis → `scientific-debugging`
- Incident isolation · resilience design → `resilience-patterns`
- Code-level performance profiling → language-specific profilers (outside observation)


## Korean Field Tools and Context

- **Commercial APM**: Datadog · New Relic · Dynatrace (mainstream at Toss · Coupang · Baemin)
- **Open source**: Prometheus + Grafana + Loki + Tempo, Elastic Stack
- **Domestic / locally hosted**: Pinpoint (Naver open source, Java APM), Scouter (LG CNS open source)
- **Korean cloud**: NHN Cloud Log & Crash, Naver Cloud Monitoring, KT Cloud Monitoring
- **Financial-sector network-segregated environments**: restrictions on bringing in external SaaS APM → many build internal systems themselves (ELK + Grafana combination). Pattern of exporting only metrics via the OpenTelemetry Collector.
- **FSS · Electronic Financial Business reporting**: mandatory incident reporting under the Electronic Financial Supervisory Regulations (previously "report without delay for outages / delays of 10 minutes or more"; relaxed in 2025 to a 30-minute threshold for services with fewer than 10,000 subscribers). Specific criteria require checking the latest regulations and enforcement rules. Reflect this regulation as an upper bound when designing MTTR · Error budget.
- **Published observability cases from Toss · Kakao Pay**: Blogs and Tech sessions use a mix of RED + USE + Golden Signals. Numerous migration cases from Pinpoint → Datadog / Prometheus.

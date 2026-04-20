---
name: twelve-factor
version: "0.2.0"
description: "Twelve-Factor App (Adam Wiggins, Heroku) — the 12 principles for cloud-native apps. Baseline hygiene for deployment, operations, and scaling. Check along 12 axes: config, dependencies, port binding, logs, processes, and more."
---

# Twelve-Factor App

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

**The 12 baseline hygiene items that make an app ready to ship to the cloud.** Most now feel like common sense, but violations are still common in 2026.


## When to Use It

- Designing or refactoring cloud-native apps
- Checklist for migrating legacy systems to the cloud
- Common vocabulary during new-hire onboarding
- Baseline check before automating deployment
- A prerequisite for moving to Kubernetes or ECS


## Applying It in Practice

### As a Checklist
Spend five minutes per factor:
- [ ] Is config in env vars?
- [ ] Do logs go to stdout?
- [ ] Are build, release, and run separated?
- [ ] Are processes stateless?
- [ ] Startup and shutdown times?

Finding one or two violations is normal. Do not rush to fix everything — *start with the ones tied to operational pain*.

### Modern Extensions ("Beyond 12 Factor", Kevin Hoffman)
- API First
- Telemetry
- Authentication & Authorization
- Self-service
- A more nuanced view of backing services


## Anti-Patterns

- **Committing config as YAML files** — a violation the moment it differs between environments
- **Storing sessions or caches in local files** — lost on process restart
- **Rotating log files inside the app** — leave that to the environment
- **Services depending on OS-specific binaries** — breaks containerization and portability
- **"SQLite in dev, Postgres in prod"** — a Parity violation


## Limitations

1. **Web/API-centric** — embedded systems, ML training jobs, and desktop apps only partly fit
2. **Limited for stateful services** — operating Kafka or a DB itself follows different principles
3. **Specific implementation details have been superseded** — by K8s, service meshes, OTel
4. **Cultural change must come with it** — without DevOps/CI/CD, 12-factor alone is not enough


## Frameworks to Pair With

- `observability` — the modern counterpart of XI (Logs)
- `resilience-patterns` — complements IX (Disposability) and VIII (Concurrency)
- `modular-monolith` — principles for internal structure
- *Combined effect*: following 12-factor slashes the cost of adopting resilience and observability later


## When This Framework Is *Wrong*

- Single-user CLIs and desktop apps
- Embedded or IoT environments with tight constraints
- Data-plane operations (running the DB or MQ itself)

---
name: twelve-factor
version: "0.1.0"
description: "Twelve-Factor App (Adam Wiggins, Heroku) — the 12 principles for cloud-native apps. Baseline hygiene for deployment, operations, and scaling. Check along 12 axes: config, dependencies, port binding, logs, processes, and more."
---

# Twelve-Factor App

## One-Line Summary

**The 12 baseline hygiene items that make an app ready to ship to the cloud.** Most now feel like common sense, but violations are still common in 2026.

## Theoretical Origin

- **Adam Wiggins** and fellow Heroku engineers (2011) — `12factor.net`.
- Distilled from running thousands of SaaS apps.
- Became the foundational assumption underlying the Kubernetes and Docker ecosystems.

## The 12 Principles

### I. Codebase
**One app = one codebase, many deploys**
- One Git repo = one app
- Multiple repos for the same app = violation (extract shared code as a library)
- Multiple apps inside a monorepo is fine

### II. Dependencies
**Declare and isolate dependencies explicitly**
- `package.json`, `requirements.txt`, `go.mod`, `Gemfile`
- No implicit reliance on system packages
- Commit lock files

### III. Config
**Put everything that differs per environment into environment variables**
- DB URL, API keys, feature flags
- Do not hard-code in the source
- 12-Factor litmus test: "Could you open-source this?" — if not, it belongs in config

### IV. Backing Services
**Treat backing services as swappable resources**
- DB, cache, MQ referenced via a single URL
- Swap local Postgres ↔ RDS with no code change

### V. Build, Release, Run
**Strictly separate the three stages**
- Build: source → executable bundle
- Release: bundle + config → release
- Run: execute the release
- No editing code in production

### VI. Processes
**Run the app as stateless processes**
- Do not use memory or local disk as persistent storage
- Keep sessions in a backing service like Redis
- Scaling = add more processes

### VII. Port Binding
**Export services via port binding**
- Self-contained, no dependency on an external web server (nginx, Apache)
- Node/Go/Flask opens the port itself and serves HTTP directly
- Routing and SSL live at a higher layer (load balancer, ingress)

### VIII. Concurrency
**Scale out via the process model**
- Horizontal (add processes) instead of vertical (bigger machine)
- Unix process model: process types such as web / worker / scheduler
- Let the OS manage processes (systemd, K8s)

### IX. Disposability
**Fast startup, graceful shutdown**
- Start in seconds
- On SIGTERM, finish current work and shut down cleanly
- Crash-safe: recovery happens via restart

### X. Dev/Prod Parity
**Keep development, staging, and production as similar as possible**
- Time gap: deploy more frequently (days → hours)
- People gap: developers deploy too
- Tools gap: dev and prod use the same DB and MQ

### XI. Logs
**Treat logs as event streams**
- The app writes to stdout/stderr
- Collection, storage, and analysis are the environment's job (Stackdriver, ELK, Loki)
- The app does not manage log files

### XII. Admin Processes
**Run admin tasks as one-off processes**
- Migrations and one-off scripts run from the same codebase and same release
- Production REPL access uses the same environment

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

## Further Reading

- **Primary source**: 12factor.net (free, short, whole text recommended)
- Hoffman, K. *Beyond the Twelve-Factor App.* (2016)
- CNCF documentation and the official Kubernetes tutorials

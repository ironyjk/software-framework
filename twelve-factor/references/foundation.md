# Twelve-Factor App -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

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


## Further Reading

- **Primary source**: 12factor.net (free, short, whole text recommended)
- Hoffman, K. *Beyond the Twelve-Factor App.* (2016)
- CNCF documentation and the official Kubernetes tutorials
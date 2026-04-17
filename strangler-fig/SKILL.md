---
name: strangler-fig
version: "0.1.0"
description: "Strangler Fig Pattern (Martin Fowler) — don't rip out legacy all at once; grow a new system around it and replace it gradually. Includes Branch by Abstraction and Parallel Change. The alternative to failed rewrites."
---

# Strangler Fig Pattern

## One-Line Summary

**Don't cut down the old tree — plant a fig tree beside it and let it slowly wrap and strangle the old one.** Don't rewrite a legacy system all at once; let a new system replace the old one feature by feature.

## Theoretical Origin

- **Martin Fowler** — "StranglerFigApplication" (2004). The analogy came from a strangler fig tree he saw while traveling in Australia.
- Alternative names: Strangler Pattern, Gradual Migration.
- Companion techniques: **Branch by Abstraction** (Paul Hammant), **Parallel Change / Expand-Contract** (Danilo Sato).
- Roots: a practical response to Joel Spolsky's 1990s argument that "rewrites almost always fail."

## Why Rewrites Fail

- The business doesn't stop (new features keep coming)
- The original system's code contains *tacit knowledge* (edge cases, history)
- Full replacement = Big Bang = no rollback if it fails
- Team morale collapses ("we've been rewriting for two years and still nothing ships")

## The Strangler Strategy

### Stages
```
1. Identify boundaries: choose the feature/module to replace (start small and independent)
2. Wrapper (Facade): a routing layer in front of the old system
3. New implementation: a new service/module provides that feature
4. Traffic migration: partial → full, gradually
5. Delete the old code
6. Repeat
```

### Companion Techniques

**Branch by Abstraction** (Hammant)
- Create an *abstract interface* on top of the legacy
- Move the existing implementation behind that interface
- Add the new implementation behind the same interface
- Switch via a flag
- Remove the old implementation

**Parallel Change (Expand-Contract)** (Sato)
- Expand: add the new version, keep the old version
- Migrate: gradually move callers and data
- Contract: remove the old version

**Feature Toggle**
- Runtime switch between new and old paths
- Canary rollout (1% → 10% → 50% → 100%)
- Instant rollback when there's a problem

## When to Use

- Legacy monolith → microservices/modular monolith transition
- Tech stack replacement (PHP→Go, JSP→React)
- Data store migration (RDB→NewSQL, single DB→sharded)
- External API version upgrade
- Codebase split (corporate spin-off, M&A)

## Field Application

### Traffic Routing Location
- **Ingress / API Gateway** — cleanest. Path/header-based routing.
- **Load Balancer** — weighted routing
- **Service Mesh** — Istio VirtualService, etc.
- **Application Layer** — branching logic inside the legacy (last resort)

### Data Migration Patterns
- **Dual Write**: write to both new and old DBs (verify consistency)
- **CDC (Change Data Capture)**: replicate changes from the old DB to the new DB
- **Outbox Pattern**: migrate via events
- Make data ownership explicit: each domain has one authoritative home

### Stage Checkpoints
- [ ] Is the boundary clearly defined?
- [ ] Canary at 1% has passed 24 hours
- [ ] Rollback procedure is automated
- [ ] Shadow traffic logs comparing old vs. new results
- [ ] SLOs are being maintained

## Anti-patterns

- **Strangler without boundaries** — that's just a slower rewrite. Without boundaries, the old system never dies.
- **Big Bang dressed as Strangler** — building 95% and flipping once = still a Big Bang
- **Never deleting the old code** — maintaining two systems = hell. Deletion is the core of the plan.
- **Same mistakes in the new system** — new code without architectural and testing discipline becomes the new legacy
- **Data migration regret** — later discovering "wait, we were still writing to the old DB?"

## Limitations

1. **Long duration** — a real strangler takes 1–5 years. Leadership patience required.
2. **Dual operating cost** — both systems must be run during migration
3. **Slow initial pace** — boundary and routing setup come first
4. **Technically infeasible in some areas** — when the data model change is very large, strangler is difficult
5. **Psychological resistance** — "if it's going to take this long, just rewrite it" keeps coming up

## Conditions for Success

1. **Leadership guarantees the long-term plan**
2. **Explicit "delete old code" milestones** — without these, permanent cohabitation
3. **The team understands both systems** — a team that only knows the new side fails
4. **Boundary-selection judgment** — the first target should be *small, independent, and visibly valuable*
5. **Automated canary and rollback**

## Context for Korean Legacy Environments

- **PHP 5.x/7.x e-commerce monoliths** — typical in 10–15 year-old large malls such as Tmon, 11st, and affiliates. Tangled MySQL FKs, triggers, and stored procedures are strangler's biggest enemy.
- **JSP + Struts + iBATIS in finance/public sector** — built in the 2000s, vendor lock-in. Due to dependencies on network separation (망분리), digital signatures, and public certificates (공인인증), strangler is more realistic than rewrite.
- **SI vendor handover code** — no docs, no tests, no owner. Before starting a strangler, *code archaeology* (git blame + interviews) is needed.
- **Public cloud migration (MOIS/KISA guidelines)** — cloud-native transition is becoming mandated. A staged approach with strangler + `twelve-factor`.
- **Financial sector "next-generation projects" (차세대 프로젝트)** — 2–3 year large-scale rebuild is conventional — still Big Bang-oriented. Recent startup models like Kakao Bank and Toss Bank have influenced incremental approaches.
- **Executive announcements of "3-year microservice transition"** — politically, no-Big-Bang is accepted, but *whether the destination must be microservices* is a separate discussion. A hybrid of modular monolith + selective microservices is the realistic proposal.

## What to Use Alongside This Framework

- `hexagonal` — apply Ports & Adapters to the new boundary
- `ddd` — theoretical basis for boundary selection (bounded context)
- `modular-monolith` — the intermediate destination before going microservice
- `observability` — new/old comparison and SLO verification during migration

## When This Framework Is *Wrong*

- Truly small systems → just rewriting is faster
- Fundamentally broken schema that's incompatible → running a new DB in parallel is required
- Regulatory or contractual "replace everything now" mandate

## Further Learning

- Fowler, M. "StranglerFigApplication" (martinfowler.com)
- Newman, S. *Monolith to Microservices.* Ch. 4.
- Hammant, P. "Branch by Abstraction" (blog)
- Sato, D. "ParallelChange" (martinfowler.com)

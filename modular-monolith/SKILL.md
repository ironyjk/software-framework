---
name: modular-monolith
version: "0.1.0"
description: "Modular Monolith — an architecture that enforces strict module boundaries within a single deployment unit. Shopify and Basecamp case studies. The benefits of boundaries without the complexity of microservices. A safe starting point for future separation."
---

# Modular Monolith

## One-Line Summary

**One deployment on the outside, microservice-like divisions on the inside**. Deployment, transactions, and debugging stay simple (the strength of the mono), while code and team boundaries remain clear (the strength of micro). This should be the *default choice* for most startups and products.

## Theoretical Origins

- **DHH / Basecamp** — "The Majestic Monolith" (Signal v. Noise blog, 2016). The origin of the term "Majestic Monolith."
- **Shopify (Kirsten Westeinde)** — "Deconstructing the Monolith" (shopify.engineering, 2019). A case study of transitioning a Rails monolith into a modular mono.
- **Simon Brown** — Popularized the term "Modular Monolith."
- **Sam Newman** — *Monolith to Microservices* (O'Reilly, 2019). A strategy for incremental separation.

## Core Principles

1. **One process, one deployment** — Preserve the simplicity of a monolith
2. **Enforce module boundaries** — Expose only externally facing APIs, hide internal implementation
3. **Shared DB allowed, shared tables forbidden** — Clear per-module schema/ownership
4. **Inter-module communication = explicit interfaces** — Function calls, but contracts at the service level
5. **Splittable later** — Boundaries such that each module can later be extracted into a separate service

## Why Before Microservices

| Item | Mono | Modular Mono | Microservices |
|---|---|---|---|
| Deployment complexity | Low | Low | High |
| Transactions | Simple | Simple | Distributed (hard) |
| Debugging | Local stack | Local stack | Distributed tracing required |
| Boundary discipline | Loose | Enforceable | Enforced |
| Team independence | Low | Medium | High |
| Operational cost | Low | Low | High |
| Independent scaling | Not possible | Not possible | Possible |

*Cost-benefit for early and mid-stage startups*: Modular Monolith wins most of the time.

## How to Enforce Module Boundaries

### Language Level
- Java: module-info.java (Java 9+)
- Rust: crate boundaries
- Python: package structure + linter (import-linter)
- TypeScript: workspace packages + lint rules
- Ruby: packwerk (Shopify open source)

### DB Level
- Separate schema per module
- External modules forbidden from reading/writing that schema
- Joins needed → call an internal API

### Code Review Level
- Reviewer principle: "this module should not know about that module"
- Circular dependency detection tooling

## When to Use

- Startup MVP ~ mid-stage product
- Team size 1–30
- Domain centered on one or two products
- No independent scaling requirements yet
- **Choose as the default**

## When to Split into Microservices (Start with Reasons Not to Split)

**Legitimate reasons** to split:
1. Strong need for independent team deployments (teams of 50+, etc.)
2. A specific module needs independent scaling (10× the load of other modules)
3. Genuinely different tech stacks/runtimes are truly required
4. Organizational separation (M&A, spinoff)

Reasons you should NOT split:
1. "Because microservices are cool" — the #1 failure reason
2. "Because the code base is big" — solve with modularization
3. "Performance issues" — profile first
4. "Because this team wants independence" — sort out boundaries first

## Real-World Application

### Shopify Case
- Retained Rails monolith, ~millions of lines
- `packwerk` inspects module boundaries
- Public APIs exist: Merchants, Products, Orders, etc.
- *Even at this scale*, a majestic monolith

### Incremental Adoption
1. Identify implicit boundaries in the current mono
2. Select module boundary candidates (bounded context ≈ module)
3. Draw the dependency graph and remove cycles
4. Move code per module + explicit APIs
5. Block boundary violations with linters/CI

## Antipatterns

- **Modular in name only** — No boundaries, still tangled
- **Shared DB, joins everywhere** — Not real boundaries
- **Too many modules** — Unmanageable past 20. Typically 5–10
- **Over-engineering "because we'll split it later"** — YAGNI
- **Premature split into micro** — A team that couldn't keep separation in the mono won't in micro either

## Limitations

1. **Single-deployment blast radius** — A deployment mistake in one module affects the whole
2. **Single-runtime constraint** — Forces the same language/framework
3. **No independent scaling** — Cannot allocate CPU/memory per module
4. **Deployment coordination cost as team grows** — Becomes a bottleneck at 20+ people

## Frameworks That Pair with This One

- `ddd` — bounded context = module
- `hexagonal` — internal structure of a module
- `team-topologies` — aligning teams with modules
- `strangler-fig` — legacy mono → modular mono transition

## When This Framework Is *Wrong*

- 100+ team members requiring independent deployment → split into micro
- Module boundaries already clear and operations mature → candidate for micro
- Extremely different tech stacks required → split

## Further Reading

- Shopify Engineering. "Deconstructing the Monolith" (blog).
- Newman, S. *Monolith to Microservices.* (2019)
- Brown, S. *Software Architecture for Developers.* — modular monolith section.
- Packwerk (github.com/Shopify/packwerk) — Ruby module boundary inspection tool.

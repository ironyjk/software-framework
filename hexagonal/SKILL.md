---
name: hexagonal
version: "0.2.0"
description: "Hexagonal Architecture (Ports & Adapters) — Alistair Cockburn. Isolate domain logic from I/O (DB, HTTP, queues, UI). Testability, replaceability, technology independence."
---

# Hexagonal Architecture (Ports & Adapters)

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

**Domain logic in the center. I/O on the outside. The two communicate only through ports (interfaces).** Even when the tech stack or framework changes, the domain stays the same.


## When to Use

- When you want to run domain logic tests *without a DB or network*, fast
- When external systems (PG companies, bank APIs) are volatile — replaceability matters
- Long-term maintained products
- Multiple entry points (HTTP + CLI + batch)


## Practical Application

### Port Design Example (Payment Domain)
```
// Primary Ports (use cases the domain "provides")
interface ChargePayment {
  execute(cmd: ChargeCommand): ChargeResult
}

// Secondary Ports (externals the domain "needs")
interface PaymentGateway {
  charge(req: ChargeRequest): GatewayResponse
}
interface OrderRepository {
  findById(id: OrderId): Order | null
  save(o: Order): void
}
```

### Test Strategy
- Unit: Domain + Fake Adapters (in-memory implementations)
- Integration: contract tests against real Adapters
- E2E: just one or two happy paths

### Gradual Adoption
New repo: from day one.
Existing repo: wrap the most volatile single domain with Adapters, then expand gradually.


## Anti-Patterns

- **Exposing Response DTOs at the Port** — if a Spring ResponseEntity shows up at the Port, it's a failure
- **Domain rules in Adapters** — "business logic in DB triggers" is the classic failure
- **Domain as ORM entity** — if a JPA @Entity doubles as the Domain class, you have tight coupling
- **Over-application**: full port/adapter stack for a simple API that's 90% CRUD → over-engineering


## Limitations

1. **Boilerplate increase** — port, adapter, and mapper layers. Overkill for small apps.
2. **Mapper cost** — ongoing maintenance of Adapter ↔ Domain conversion code
3. **Team learning curve** — requires discipline to maintain boundaries. Weak teams may become more confused.
4. **Fad-driven adoption** — starting with "let's use clean architecture" frequently fails. It must be justified by pain.


## When *Not* to Use

- Short feedback-loop MVP stages
- Thin CRUD-centric services
- Teams of fewer than 3 members with expected product lifespan under 6 months


## When This Framework *Needs Complementing*

- High domain complexity → `ddd` (Hexagonal handles boundaries, DDD handles the model)
- Financial ledgers or state reconstruction needs → `event-sourcing-cqrs`
- Code-level dependency management → `solid`

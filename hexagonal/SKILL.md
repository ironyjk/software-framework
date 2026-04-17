---
name: hexagonal
version: "0.1.0"
description: "Hexagonal Architecture (Ports & Adapters) — Alistair Cockburn. Isolate domain logic from I/O (DB, HTTP, queues, UI). Testability, replaceability, technology independence."
---

# Hexagonal Architecture (Ports & Adapters)

## One-Line Summary

**Domain logic in the center. I/O on the outside. The two communicate only through ports (interfaces).** Even when the tech stack or framework changes, the domain stays the same.

## Theoretical Origins

- **Alistair Cockburn** — "Hexagonal Architecture" (2005, alistair.cockburn.us). The original idea was formulated during the Crystal methodology days in the 1990s.
- Similar concepts:
  - **Onion Architecture** — Jeffrey Palermo's blog series (2008). Stricter dependency direction.
  - **Clean Architecture** — Robert Martin's *Clean Architecture* (2017). Onion + explicit Use Case layer.
  - **Ports and Adapters** — Cockburn's preferred term since 2008 (Hexagonal is just a metaphor).
- All three share the same core idea: *domain in the center, dependencies pointing only inward*.
- Purpose: prevent "spinning up a DB just to test," secure the possibility of framework replacement.

## Structure

```
           ┌─────────────────────┐
   Driving │  Primary Adapters   │   (HTTP, CLI, gRPC, Scheduler)
   side    │  (inbound)          │
           └─────────┬───────────┘
                     │ Primary Port
           ┌─────────▼───────────┐
           │    Domain Core      │   ← business logic lives only here
           │  (entities·rules)   │
           └─────────┬───────────┘
                     │ Secondary Port
           ┌─────────▼───────────┐
   Driven  │  Secondary Adapters │   (DB, External API, MQ, Cache)
   side    │  (outbound)         │
           └─────────────────────┘
```

### Terminology

- **Port** — the interface through which the domain interacts with the outside. Defined in domain language.
- **Adapter** — an implementation of a port using a specific technology (PostgreSQL, Kafka, REST).
- **Driving (Primary)** — the side that *calls* the domain from the outside (HTTP controllers, CLI).
- **Driven (Secondary)** — the side the domain *commands* outward (Repository, EventPublisher).

## Core Rules

1. **One-way dependency direction**: Adapter → Port → Domain. *The Domain does not know about Adapters*.
2. **Ports are owned by the domain**: Use domain vocabulary (Order, Customer), not DB vocabulary (row, table).
3. **Prevent framework infiltration**: if Spring/Django annotations or base classes seep into the Domain, it's a failure.
4. **Adapters are replaceable**: swapping PostgreSQL → MongoDB is just an adapter swap.

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

## Further Learning

- Cockburn, A. "Hexagonal Architecture." (original, 2005)
- Martin, R. *Clean Architecture.* (similar concept)
- Vernon, V. *Implementing Domain-Driven Design.* Ch. 4.

# Hexagonal Architecture (Ports & Adapters) -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

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


## Further Learning

- Cockburn, A. "Hexagonal Architecture." (original, 2005)
- Martin, R. *Clean Architecture.* (similar concept)
- Vernon, V. *Implementing Domain-Driven Design.* Ch. 4.
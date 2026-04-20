# Strangler Fig Pattern -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

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


## Further Learning

- Fowler, M. "StranglerFigApplication" (martinfowler.com)
- Newman, S. *Monolith to Microservices.* Ch. 4.
- Hammant, P. "Branch by Abstraction" (blog)
- Sato, D. "ParallelChange" (martinfowler.com)
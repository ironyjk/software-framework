---
name: scientific-debugging
version: "0.2.0"
description: "Scientific Debugging — hypothesis-driven debugging. Treat bugs not as mysteries but as a chain of falsifiable hypotheses. Observation→Hypothesis→Experiment→Verification loop. Based on Zeller's *Why Programs Fail*."
---

# Scientific Debugging

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

A bug is a **chain of falsifiable hypotheses**. Observe → hypothesize → predict → experiment → verify → fix and repeat. Don't dig in based on "gut feeling" — always be conscious of *what the next experiment will rule out*.


## 12 Core Habits

1. **Reproduce first** — If you can't reproduce, all other debugging is guesswork
2. **Minimal reproduction** — Reduce input until removing it makes the symptom disappear (→ delta debugging)
3. **One variable at a time** — Change only one variable per experiment
4. **Document expectations** — Before running, write down "this should yield X" and compare with reality
5. **Suspect the code** — Suspecting framework/compiler bugs comes last
6. **Recent changes first** — Regressions come from the recent diff
7. **Don't hesitate to add more logging** — Visibility lowers hypothesis cost
8. **Use stack and state dumps** — Memory snapshots, heap dumps
9. **Off-by-one checklist** — Boundary values (0, 1, max, null, empty, negative)
10. **Rubber duck** — Explaining to someone/something = clarifying the hypothesis
11. **Debugger ≠ always required** — Sometimes print + logs are faster
12. **After the fix, "why did it happen?"** — Document the root cause after fixing (5 Whys)


## When to Use

- Tracing the cause of a reproducible bug
- Understanding "why does this system behave this way"
- Investigating unexpected behavior
- Exploring the cause of performance degradation (together with `observability`)
- Narrowing the conditions of intermittent failures


## Practical Application

### Hypothesis Tree Example
```
Symptom: Payments intermittently fail
├─ H1: External bank API timeout
│   Experiment: Measure retry logs and response times
│   → Falsified (API responses normal)
├─ H2: DB connection pool exhaustion
│   Experiment: Check pool metrics
│   → Retained (pool utilization 95%+)
│   ├─ H2.1: Queries have slowed down
│   │   Experiment: slow query log
│   │   → Retained
│   └─ H2.2: Connection leak
│       Experiment: Search for missing explicit close
│       → Falsified
```

### Key: *Start with experiments that have the lowest falsification cost*
Example: log grep < metric dashboard < rebuilding reproduction environment < production A/B


## Anti-Patterns

- **Shotgun debugging** — "Let's just change anything"
- **Heisenberg** — Adding print/logging changes the bug itself (common in race conditions)
- **Fix-forward panic** — Firing off hot fixes before finding the root cause
- **Confirmation bias** — Collecting only evidence that *confirms* your hypothesis
- **Premature abstraction** — Changing structure without knowing the cause


## Limitations

1. **Non-reproducible bugs** — In flaky tests and race conditions, the cost of verifying hypotheses skyrockets
2. **Distributed systems** — Cause decomposition is not as clean as on a single machine (→ `observability`)
3. **Time pressure** — During service outages, sometimes "stopgap first, analysis later" is chosen
4. **Cognitive bias** — Hypothesis diversity drops in familiar code


## When This Framework Is *Wrong*

- Need to trace the failing commit → `bisection`
- Observing distributed systems → `observability`
- Preventing cascading failures → `resilience-patterns`

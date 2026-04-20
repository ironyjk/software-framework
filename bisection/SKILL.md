---
name: bisection
version: "0.2.0"
description: "Bisection — git bisect, binary search, delta debugging. Isolate 'one cause among N candidates' in log N steps. Regression hunting, outage window narrowing, minimal reproducer input discovery."
---

# Bisection

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

**Binary search when the cause space is large and deterministic**. Find the culprit among N commits, N inputs, or N configurations in `log₂ N` steps.


## Three Variants

### 1. Commit Bisection (git bisect)
- "Since when has it been broken?"
- Commands: `git bisect start / bad / good <ref>`
- At each step, judge `good` or `bad` → narrow down
- Automation: `git bisect run ./test.sh` (exit 0 = good, non-zero = bad)

### 2. Binary Search Input
- "Which input triggers it?"
- Isolate a single problem from massive request logs or test suites
- Remove halves of the original input and check failure reproduction

### 3. Delta Debugging
- "The *minimal* input required to reproduce the failure"
- Automated minimization algorithm (ddmin)
- HTML parser crash → Zeller's case of reducing a 3000-line page to 12 characters


## When to Use

- When there is a regression and both good and bad points are clear
- Tests must be deterministic
- Discrete search space: logs, inputs, flags, library versions, etc.


## Preconditions

- **Determinism** — Same conditions yield the same result. Flaky tests break bisect.
- **Monotonicity** — Always bad after some point (no temporary recovery)
- **Automatable judgment** — Manual judgment makes each step costly

If non-deterministic: run multiple times with majority voting, or switch to `scientific-debugging`.


## Practical Application

### Standard git bisect Routine
```
git bisect start
git bisect bad HEAD
git bisect good v1.4.2        # last known good release
git bisect run npm test       # automated execution
# Found the culprit commit in 10 steps out of 1000 commits
git bisect reset
```

### Judgment Script Tips
- exit 0 = good, 1~124/126~127 = bad, 125 = skip (e.g., build failure)
- For unintended commits (unbuildable), use `git bisect skip`

### Log Bisection Example
```
Which of 3000 failing requests is the cause?
→ Run batch 1~1500 → failure reproduced? Y → 1~750 → ...
→ Isolated to 1 request in 10 rounds
```

### Delta Debugging Tools
- `creduce` (C/C++)
- `shrinker` (built into property-based tests)
- Python: `pysearch`, custom scripts


## When Not to Use

- **Non-deterministic failures** — race conditions (→ `scientific-debugging`)
- **Broken monotonicity** — bug appears in commit A, coincidentally hidden in B, reappears in C
- **State dependency** — commit ranges that include DB migrations (each step needs DB reset)
- **Small candidate set** — fewer than 5 candidates is faster to check sequentially


## Antipatterns

- **Bisecting without tests** — manual judgment causes frequent misjudgments
- **Bisecting with flaky tests** — misjudging `good` as `bad` contaminates the whole run
- **Bisecting with integration tests** — slow and environment-dependent. Reproduce at the unit level if possible
- **Resting on laurels after bisect** — finding the culprit commit is not root cause analysis


## Limitations

1. **Large merge commits** — if a merge commit is the culprit, its interior needs another bisect
2. **Infrastructure changes** — environmental changes outside of commits (libraries, OS) are not reflected
3. **Dependency reinstall cost** — across ranges with node_modules/lockfile changes, each step takes minutes


## When This Framework Is *Wrong*

- Non-deterministic or intermittent failures → `scientific-debugging`
- Anomalies in a running distributed system → `observability`
- Designing to contain outages → `resilience-patterns`

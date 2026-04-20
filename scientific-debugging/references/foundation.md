# Scientific Debugging -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

## Theoretical Origins

- **Andreas Zeller** — *Why Programs Fail: A Guide to Systematic Debugging* (2005)
- **David Agans** — *Debugging: 9 Indispensable Rules* (2002)
- Applying the scientific method (Popper) to software. "Falsification over confirmation."


## Core Loop

```
1. Observation    — Describe the failure symptom precisely
2. Hypothesis     — Candidate cause (must be testable)
3. Prediction     — If the hypothesis is true, X will be observed
4. Experiment     — Test the prediction at minimum cost
5. Observation    — Prediction vs. reality
6. Keep/Discard and repeat — Narrow down the hypothesis tree
```


## Further Learning

- Zeller, A. *Why Programs Fail.*
- Agans, D. *Debugging.* (a quick-reading textbook)
- Allspaw, J. *Blameless PostMortems and a Just Culture.* (web)
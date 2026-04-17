# Software Framework

[한국어](README.ko.md) | **English**

---

A software engineering meta-router + framework collection. Bundles 12 frameworks — debugging, architecture, design, resilience, evolution, and team structure — under a single routing layer.

## Design Principles

1. **Identifiable frameworks, not vague practices** — Tools where "what to look at" is clear, not just "do it well."
2. **Timeless principles + current practice** — Classics like Fowler, Evans, Cockburn combined with modern field practice: Team Topologies, Resilience Patterns, etc.
3. **Anti-pattern warnings included** — Explicit notes on when each framework misleads or causes harm.

## Structure

```
software-framework/
├── SKILL.md                # Meta-router
├── scientific-debugging/   # Hypothesis-driven debugging
├── bisection/              # git bisect · binary search · delta debugging
├── observability/          # USE + RED + 4 Golden Signals
├── hexagonal/              # Ports & Adapters (Cockburn)
├── ddd/                    # Domain-Driven Design (Evans)
├── event-sourcing-cqrs/    # Greg Young
├── modular-monolith/       # Shopify / Basecamp style
├── solid/                  # Robert Martin's 5 principles
├── twelve-factor/          # Heroku 12-Factor App
├── resilience-patterns/    # CB · bulkhead · back-pressure · rate-limit
├── strangler-fig/          # Fowler legacy replacement
└── team-topologies/        # Skelton & Pais + Conway
```

## Usage

```
/code <situation description>
```

The meta-router maps problem signals and selects 1–3 frameworks, then executes and synthesizes via the Skill tool.

> 💡 **Short name**: after [wrapper setup](https://github.com/ironyjk/claude-frameworks-marketplace#short-command-setup-optional), callable as `/code`

## Out of Scope

- Language- or runtime-specific best practices
- Framework selection (React vs. Vue, Django vs. Rails, etc.)
- Business / product strategy (→ `think`)
- Negotiation / communication (→ `howtotalk`)

## Meta

- `last_verified: 2026-04-17`
- `valid_until: 2027-04-17` — Architecture and design principles have long lifespans; 1-year re-verification cycle. Team Topologies and Resilience Patterns items may be refreshed earlier if practice shifts.

## Disclaimer

This repo is a design-support tool; it does not substitute for choosing a specific tech stack, vendor, or framework. Real systems involve trade-offs that vary by team size, domain, and regulations (e.g., Korean FSS electronic-finance requirements).

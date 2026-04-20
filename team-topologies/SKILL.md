---
name: team-topologies
version: "0.2.0"
description: "Team Topologies (Skelton & Pais) — 4 team types + 3 interaction modes + cognitive load + Inverse Conway Maneuver. Team structure shapes architecture. For team scaling, platformization, and microservices organization design."
---

# Team Topologies

> **Background and theory**: Read [references/foundation.md](references/foundation.md)


## One-Line Summary

**Team structure determines software structure (Conway's Law).** So if you want a specific architecture, organize your teams accordingly first (Inverse Conway Maneuver). Explicitly design your organization using 4 team types and 3 interaction modes.


## When to Use

- When reorganizing as the organization crosses 20, 50, or 200-person tiers
- When designing teams during a monolith → microservices transition
- When establishing a new platform engineering organization
- Diagnosing slowdowns or increased waiting time between teams
- Assessing DevOps maturity stage


## Practical Application

### Team Composition Checklist
- [ ] **2-8 members** per team (Amazon 2-pizza team)
- [ ] One team handles **one value stream**
- [ ] The **number of dependencies** between teams is explicitly visible (ideally 3-4 or fewer)
- [ ] Each team's **cognitive load** is manageable
- [ ] If a Platform Team exists, is it operated as "Platform as a Product"?

### Inverse Conway Maneuver Example

**Target architecture**: A modular monolith + microservices where payment, orders, and inventory deploy independently

**Organizational restructuring**:
- Payment Stream-Aligned Team
- Orders Stream-Aligned Team
- Inventory Stream-Aligned Team
- Platform Team (CI/CD, observability, DB as a Service)
- Payment Core Complicated Subsystem Team (PCI, authentication, settlement logic)
- Enabling Team if needed (SRE diffusion)

→ After 6 months of operation, service boundaries naturally form *following team boundaries*.

### Team API

Each team publishes:
- Owned systems and services
- How to engage with the team (interaction modes, SLAs)
- Roadmap and current priorities
- Contact info, on-call information


## Anti-Patterns

- **Functional team silos** — "Frontend team / Backend team / DB team" disrupts streams
- **"Throw issues at the Infra team"** — Ticket hell instead of X-as-a-Service
- **When Platform isn't a product** — If teams don't want to use it, it's not a platform
- **All teams in collaboration mode** — Collaboration fatigue accumulates
- **Microservices without team boundaries** — Fails via the inverse of Conway
- **Complicated Subsystem team becomes a black box** — Becomes a bottleneck without communication


## Limitations

1. **Involves organizational politics** — Team reorganization raises power and budget issues
2. **Dunbar constraint** — In 50+ person organizations, grasping all interactions has limits
3. **Overkill in early startups** — At the 5-15 person stage where the team is the whole org, it's all one team
4. **Trap of fixed classification** — Types and modes must evolve as teams change
5. **The book isn't a solution** — It's a diagnostic tool + vocabulary. Application requires context.


## Common Failures in Korean Organizational Context

- **Functional silos like "Platform Division / Development Division"** — A common early-stage trap for Toss, Naver, and Kakao. Absence of Stream-aligned teams → every project requires 3-4 division handoffs.
- **"TF (Task Force) overuse"** — Failing to draw permanent team boundaries and substituting with TFs → ambiguous belonging and diffused responsibility. Stream-aligned teams must be a *permanent structure*.
- **Wide ownership by founding members and seniors** — If someone "knows modules A, B, and C," applying Inverse Conway triggers political resistance. Interviews and relocation plans must come first.
- **"Platform Office (Platform Team) as a ticket queue"** — Platform as a Product failure. Without self-service APIs, documentation, and SLAs, it becomes a bottleneck. Many cases at Toss and Coupang.
- **Excessive team lead / part lead hierarchy** — Korean large companies have TL → Part Lead → Team Lead → Office Head → Division Head. Without a decision delegation matrix, every decision escalates upward.
- **Compensation and evaluation systems reinforce functional silos** — "Frontend is evaluated by the frontend team" hinders stream-aligned formation. HR and evaluation systems must be designed concurrently.
- **Finalizing the org chart first and slotting people in** — The Korean-style "HR announcement" conflicts with Team Topologies. The order should be: team design → gathering intentions → finalization.


## Frameworks to Use Alongside This

- `modular-monolith` — Module boundaries = team boundaries (Conway)
- `ddd` — bounded context = stream-aligned team scope
- `strangler-fig` — Links team reorganization with legacy decomposition
- `observability` — A flagship service for Platform Teams


## When This Framework Is *Wrong*

- Organizations of 10 or fewer — A single team suffices
- Extremely highly specialized groups (R&D, research) — A fluid organization may work better
- When you lack organizational design authority and say "we should do this" — Can't push through without politics

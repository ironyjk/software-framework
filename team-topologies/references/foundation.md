# Team Topologies -- Background & Theory

> This document contains the theoretical foundations, evidence base, and references for this framework.
> For practical application and execution, see [SKILL.md](../SKILL.md).

## Theoretical Origins

- **Matthew Skelton & Manuel Pais** — *Team Topologies* (2019). A consolidation of DevOps and agile organizational design.
- Roots: Conway's Law (1968), Spotify Model, Dunbar's number.
- Modern impact: Many tech companies have adopted it for platform engineering organizational design.


## Conway's Law (The Premise)

> "Any organization that designs a system will produce a design whose structure is a copy of the organization's communication structure." — Melvin Conway (1968)

**Inverse proposition (Inverse Conway Maneuver)**: If you want a specific architecture, organize your teams into that structure first.


## The 4 Team Types

### 1. Stream-Aligned Team
- A team **aligned to a single value stream**
- End-to-end responsibility (design, development, deployment, operations)
- *Most* of the product should reside here (70-80%)
- Examples: "Payment team," "Search team," "Mobile iOS team"

### 2. Enabling Team
- Temporary **coaching and mentoring**
- Helps Stream-aligned teams acquire new skills and practices
- Does not execute itself (it teaches)
- Examples: Spreading SRE practices, supporting DDD adoption

### 3. Complicated Subsystem Team
- Areas requiring **deep expertise**
- ML, video processing, payment core, cryptography, etc.
- Reduces the load on Stream-aligned teams
- Kept small (5-9 members)

### 4. Platform Team
- Provides **internal platforms**
- CI/CD, observability, infrastructure, shared libraries
- "Platform as a Product" — internal user satisfaction is the KPI
- Designed so Stream-aligned teams use it in a *self-service* manner


## The 3 Interaction Modes

### Collaboration
- Two teams work closely together, with blurred boundaries
- When exploring new technology or complex problems
- High-cost. Short-term only.

### X-as-a-Service
- One team provides a "service," another team "consumes" it
- Clear interfaces and contracts
- The lowest-cost and most sustainable mode
- Default mode for Platform Teams

### Facilitating
- A relationship where an Enabling Team helps another team
- Temporary, teaching-centered

Inter-team relationships must always be *explicitly* one of these three. "Just collaborating occasionally" = ambiguous = friction.


## Cognitive Load

- There's a limit to the complexity a team can handle
- 3 types of load: intrinsic (essence of the domain) / extraneous (bad tooling) / germane (learning activities)
- **Eliminate extraneous load** and leave room for germane load
- If a team's scope is too broad → split it, have Platform or Complicated Subsystem teams absorb parts


## Further Learning

- Skelton, M. & Pais, M. *Team Topologies.* (The original, short and practical)
- teamtopologies.com — Official resources and diagrams
- Forsgren, N. et al. *Accelerate.* — Connects DORA metrics to team design
- Kim, G. et al. *The Phoenix Project* & *The DevOps Handbook.* — Theoretical DevOps organizational background
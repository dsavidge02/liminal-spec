# Product Research

**Purpose:** Transform a vision or rough idea into a Product Requirements Document (PRD) that scopes one or more features.

This phase is optional. Most work starts at Phase 2 (Epic) when the feature is already known.

Use Product Research when you need to:
- Explore product direction before committing to implementation.
- Align multiple stakeholders on what to build next.
- Scope several related features under one initiative.

Skip Product Research when:
- The feature is already defined.
- Scope is narrow and clear.
- You can go directly into a detailed Epic.

---

## Inputs and Outputs

| Item | Description |
|------|-------------|
| Input | Vision, problem statement, opportunity, or initiative idea |
| Output | PRD with feature candidates, scope boundaries, and success metrics |

The PRD is directional context. It informs Phase 2 but does not replace a detailed Epic.

---

## Workflow

### 1. Frame the Problem

Establish why this initiative exists:
- What user/business problem are we solving?
- Why now?
- What happens if we do nothing?

### 2. Define Users and Context

Identify the primary users and their working context:
- Primary persona(s)
- Current workflow and pain points
- Operational constraints

### 3. Draft Feature Candidates

Propose candidate features that solve the problem:
- Feature name
- Persona served
- High-level behavior
- Priority (must/should/nice)

### 4. Set Scope Boundaries

Make boundaries explicit:
- In scope
- Out of scope
- Assumptions and dependencies
- Key risks and open questions

### 5. Define Success Metrics

Specify measurable outcomes:
- User outcomes
- Business outcomes
- Operational/quality outcomes

### 6. Produce the PRD

Package the work into a single PRD artifact that can be reviewed and handed into Phase 2.

---

## PRD Template

```markdown
# Product Requirements Document: [Initiative Name]

## Vision
What problem are we solving? Why now?

## User Personas
Who are we building for? What are their goals and constraints?

## Initiative Scope

### In Scope
- [Capability/area]
- [Capability/area]

### Out of Scope
- [Excluded capability]
- [Future phase item]

## Feature Overview

### Feature 1: [Name]
**Persona:** [Who uses this]
**Summary:** [One paragraph]
**Key Flows:** [Bullets]
**High-Level ACs:** [Bullets]
**Priority:** Must-have | Should-have | Nice-to-have

### Feature 2: [Name]
...

## Architectural/Operational Considerations
Cross-cutting constraints, integration points, compliance, performance, security.

## Success Metrics
How we know this initiative worked.

## Risks and Open Questions
What could derail delivery and what must be resolved before implementation.
```

---

## PRD vs Epic

| PRD (Phase 1) | Epic (Phase 2) |
|---------------|----------------|
| Multi-feature initiative context | Single feature implementation contract |
| High-level acceptance outcomes | Detailed ACs + test conditions |
| Product alignment artifact | Engineering-ready requirements artifact |
| Directional | Traceable and testable |

---

## Handoff to Phase 2 (Epic)

For each feature selected for implementation:
1. Extract the feature from the PRD.
2. Expand it into a full Epic with AC/TC traceability.
3. Treat the Epic as the implementation source of truth.

The PRD remains context for "why"; the Epic governs "what must be built."

---

## Guidance for This Session

When running this phase:
- Ask enough questions to disambiguate goals, users, and boundaries.
- Keep recommendations concrete and prioritized.
- Call out assumptions explicitly.
- End with a complete PRD draft, ready for review.

If the user already has a clearly defined feature, recommend skipping to Phase 2 (`/ls-epic`).

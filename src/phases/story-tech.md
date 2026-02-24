# Story Technical Enrichment

**Purpose:** Add technical implementation sections to functional stories. The Tech Lead takes stories with full acceptance criteria and enriches them with implementation targets, test mapping, verification criteria, and deviation tracking.

The output is a complete story -- functional sections authored by the BA/SM, technical sections authored by the Tech Lead. Two distinct halves, different owners, different sign-off authorities. The PO accepts from the functional half. The Tech Lead signs off from the technical half. The engineer serves both.

---

## Role: Tech Lead as Enricher and Validator

The Tech Lead operates in two capacities during this phase.

**As validator:** Before adding technical sections, verify that functional stories are complete and coherent. Can you identify implementation targets for every AC? Do stories align with the tech design's module boundaries? Are there functional gaps that would block technical enrichment?

If issues found, return to BA/SM for revision. Don't enrich incomplete stories.

**As enricher:** Once validated, add technical sections below the functional Definition of Done. These sections describe *what* to implement and *how to verify* -- not step-by-step instructions for *how* to implement. The engineer brings judgment and plan mode to the "how."

---

## Inputs

Technical enrichment draws from three sources:

| Source | What It Provides |
|--------|-----------------|
| **Functional stories** | Scope boundaries, ACs, TCs, error paths, functional DoD |
| **Tech Design** | Architecture, interfaces, module boundaries, TC-to-test mapping, verification scripts |
| **Epic** | Functional requirements, data contracts, user flows |

The functional stories tell you *which* ACs are in scope for each story. The tech design tells you *how* those ACs map to modules, interfaces, and tests. The epic provides the broader functional context when stories need clarification.

---

## The Technical Sections

Below the functional Definition of Done, add these sections. Together they form the technical half of the story.

### Architecture Context

Brief orientation -- what modules, components, or subsystems this story touches. Not a repeat of the tech design; a focused extract relevant to this story's scope. Enough for an engineer to know where to look before reading the full tech design.

```markdown
## Technical Implementation

### Architecture Context

This story covers the session creation route and Convex persistence layer.
Key modules: `src/routes/sessions.ts`, `src/services/session.service.ts`,
`src/clients/convex.client.ts`.

The session creation flow follows: Route handler -> service orchestration ->
Convex persistence -> response formatting. See Tech Design Flow 1 for the
full sequence diagram.
```

### Interfaces & Contracts

The specific interfaces from the tech design that this story implements or consumes. Not all interfaces -- just the ones relevant to this story's ACs. Include enough detail that an engineer knows the contract without reading the entire tech design.

```markdown
### Interfaces & Contracts

**Creates:**
- `SessionService.create(params: CreateSessionParams): Promise<Session>`
- Route handler: `POST /sessions` -> 201 with session ID

**Consumes:**
- `ConvexClient.insertSession(session: SessionRecord): Promise<Id<"sessions">>`

**Types (from Story 0):**
- `CreateSessionParams`, `Session`, `SessionRecord`
```

### TC to Test Mapping

The critical traceability link. Every TC from this story's functional section maps to a test approach. This section is derived from the tech design's TC-to-test mapping table, filtered to this story's scope.

```markdown
### TC -> Test Mapping

| TC | Test File | Test Description | Approach |
|----|-----------|------------------|----------|
| TC-1.1a | sessions.test.ts | creates session with valid params | Service mock: mock Convex client, assert route returns 201 |
| TC-1.1b | sessions.test.ts | rejects invalid session params | Service mock: send malformed body, assert 400 |
| TC-1.2a | sessions.test.ts | returns session ID in response | Service mock: verify response shape |
```

Every TC must appear in this table. If a TC has no test mapping, either the TC is untestable (return to BA/SM) or the tech design is missing a test boundary (return to Tech Lead self-review).

### Risks & Constraints

Implementation risks specific to this story. Not a general risk register -- specific things that could go wrong or require extra care during implementation.

```markdown
### Risks & Constraints

- Convex client connection timeout: fail fast with 503, do not retry in the request path
- Session ID generation must be collision-resistant (use CUID2, not UUID v4)
- This story does NOT implement session expiry -- that's Story 3
```

### Spec Deviation Field

A required field that documents any divergence between what the tech design specifies and what the story actually implements. Most stories have no deviations -- the field is still required, explicitly empty.

```markdown
### Spec Deviation

None. Story implementation aligns with Tech Design sections 3.1 and 3.2.
```

Or when deviations exist:

```markdown
### Spec Deviation

- Tech Design specifies Redis caching for session lookups. Deferred to Story 3
  per dependency sequencing. Session reads in this story go directly to Convex.
- Tech Design Flow 1 shows auth middleware. Auth is not in scope for Story 1;
  added in Story 2. Route is unprotected in this story.
```

Spec deviation is not a failure -- it's a transparency mechanism. The engineer and verifier need to know where this story's implementation intentionally differs from the tech design, and why.

### Technical Definition of Done

Concrete verification steps. Not "make sure it works" -- specific commands and expectations.

```markdown
## Technical Checklist

- [ ] All TCs have passing tests
- [ ] TypeScript compiles clean (`bun run typecheck`)
- [ ] Lint/format passes (`bun run lint && bun run format:check`)
- [ ] No regressions on prior stories (`bun test`)
- [ ] Verification: `bun run verify`
- [ ] Spec deviations documented (if any)
```

---

## Story Contract Requirements

Every story that completes technical enrichment must satisfy four non-negotiable requirements. These are the quality gate for this phase.

**1. TC to test mapping present.** Every TC in the story's functional section has a corresponding entry in the TC-to-test mapping table. Exception: Story 0 has no TCs (it's foundation setup), so it has no TC mapping. Story 0 gets simplified technical sections instead (see below).

**2. Technical DoD present.** The technical checklist includes specific verification commands and regression expectations. Not "tests pass" -- which tests, which commands, what counts as passing.

**3. Spec deviation field present.** Every story has a spec deviation field, even when empty. An empty field means "I checked and there are no deviations." A missing field means "I didn't check."

**4. Technical sections describe targets, not steps.** The Architecture Context names modules and flows. The Interfaces section names contracts. Neither prescribes the implementation sequence. The engineer decides how to get there -- the story describes where "there" is.

---

## Story 0: Technical Enrichment

Story 0 is foundation setup -- types, fixtures, error classes, project config. It has no TCs from the epic because it delivers no user-facing functionality. Technical enrichment for Story 0 is simplified:

```markdown
## Technical Implementation

### Architecture Context

Foundation setup for [feature name]. Creates type definitions, error classes,
test fixtures, and project configuration that all subsequent stories depend on.

### Types to Create

- `CreateSessionParams` (from Tech Design Low Altitude)
- `Session`, `SessionRecord` (from Tech Design Low Altitude)
- `SessionError` extends `AppError` (from Tech Design error contract)

### Config to Validate

- Test runner configured and passing empty suite
- Path aliases resolving
- Environment variables documented in `.env.example`

### Spec Deviation

None.

## Technical Checklist

- [ ] All type definitions from tech design created
- [ ] Error classes available
- [ ] Test fixtures match data contracts
- [ ] TypeScript compiles clean
- [ ] Project config validated
```

No TC-to-test mapping (no TCs). No Interfaces & Contracts section (types are the deliverable, not consumers of interfaces). The technical enrichment is lighter because the story's scope is lighter.

---

## The Consumer Gate

The ultimate validation for technically enriched stories: **Could an engineer implement from this story without asking clarifying questions?**

This is not a rhetorical question. Apply it literally:

- Does the story tell the engineer which modules to create or modify?
- Does the TC-to-test mapping tell them what tests to write?
- Does the technical DoD tell them how to verify their work?
- Does the spec deviation field tell them where this story intentionally diverges from the tech design?
- Are there ambiguities that would force the engineer to choose between interpretations?

If the answer to the last question is "yes," the story needs more detail in the technical sections. If the engineer would need to read the full tech design to understand what to build, the Architecture Context is too thin. If the engineer would need to derive test approaches from scratch, the TC-to-test mapping is incomplete.

The consumer gate is the quality bar. Stories that pass it are ready for implementation.

---

## Coherence Check Against Tech Design

After enriching all stories, verify completeness against the tech design:

**Interface coverage:** Every interface defined in the tech design's Low Altitude section should appear in at least one story's Interfaces & Contracts section. An interface with no story assignment is either dead code in the design or a gap in the story breakdown.

**Module coverage:** Every module from the tech design's Module Responsibility Matrix should appear in at least one story's Architecture Context. A module with no story is either unnecessary or missing from the sharding.

**Test mapping coverage:** The union of all stories' TC-to-test mapping tables should cover every TC from the epic. Compare against the coverage gate table from the functional story sharding phase.

Build a cross-story coverage summary:

| Tech Design Element | Type | Story | Notes |
|---|---|---|---|
| SessionService | Module | Story 1, Story 2 | Created in S1, extended in S2 |
| ConvexClient | Module | Story 1 | |
| AuthMiddleware | Module | Story 2 | |
| Session type | Interface | Story 0 | Foundation |
| ... | ... | ... | ... |

Gaps in this table are blockers. Every element in the tech design should have a home.

---

## Common Failure Modes

**Steps masquerading as targets.** The technical section says "1. Create the route file. 2. Add the handler function. 3. Wire to the service." This prescribes implementation sequence. Instead: "Route handler at POST /sessions delegates to SessionService.create(). Returns 201 with session ID." The engineer decides the sequence.

**Missing TC mappings.** The functional section has 8 TCs but the mapping table has 5 entries. Three TCs have no test approach. This either means the tech design doesn't cover them (design gap) or the enrichment was hasty (enrichment gap). Either way, the story isn't ready.

**Vague DoD.** "Tests pass" is not a technical DoD. Which verification command? What does "pass" mean -- zero failures, or zero failures in this story's test files? Does "no regressions" mean running the full suite or just prior stories? Specificity prevents ambiguity during verification.

**Rubber-stamped spec deviation.** Every story says "None." without evidence of checking. The spec deviation field exists to force the Tech Lead to compare each story against the tech design and explicitly document alignment or divergence. If every story is "None." with no reference to which tech design sections were checked, the field is being rubber-stamped rather than used.

**Over-prescribing Story 0.** Story 0 doesn't need TC-to-test mapping because it has no TCs. Adding artificial "test the types compile" TCs to Story 0 creates busy work. The exit criteria are simpler: types exist, fixtures exist, config works, it compiles.

---

## Output: Complete Stories

After technical enrichment, each story contains:

**Functional half (BA/SM authored, PO accepts):**
- Objective, Scope, Dependencies
- Acceptance Criteria with full TCs
- Error Paths
- Definition of Done (functional)

**Technical half (Tech Lead authored, Tech Lead signs off):**
- Architecture Context
- Interfaces & Contracts
- TC to Test Mapping
- Risks & Constraints
- Spec Deviation
- Technical Checklist

These complete stories are the sole implementation artifact. Engineers receive them, form an implementation plan, and execute. No prompt packs, no orchestration scripts, no intermediate artifacts between the story and the code.

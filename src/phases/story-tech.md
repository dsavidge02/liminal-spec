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

## Tech Design Sharding

Just as ls-story shards the epic's ACs and TCs into stories, ls-story-tech shards the tech design into stories. Each story's technical section contains the relevant portion of the tech design for that story's scope -- flows, module targets, interfaces, and test plan.

The enricher's job is **curation, not compression.** Select the tech design sections relevant to this story and include them substantially. Do not summarize 500 lines of tech design into 30 lines of abstract. If the tech design has 300 lines of content relevant to this story, the technical section should carry that detail forward at roughly that scale -- edited for story scope, not condensed for brevity.

Tech design content is repeated in stories by design. This is the same pattern as ACs/TCs being repeated from the epic into individual stories. The tradeoff is intentional: accept duplication to ensure the implementer has everything needed without reading the full tech design. The implementer should be able to execute from the story alone. The tech design is available as fallback for rationale, alternatives-considered, or cross-cutting context -- not as required reading.

**Scope bound:** Include everything needed for precise execution of this story and not much more. The bound is story scope, not "everything in the tech design." When selecting content, also check for cross-cutting tech design content relevant to this story that may live outside the obvious story-scoped sections (shared contracts, error shapes, invariants).

---

## The Technical Sections

Below the functional Definition of Done, add these sections. Together they form the technical half of the story.

### Architecture Context

Story-scoped shard of the tech design's architecture. Include the relevant flows, module responsibilities, and key decisions for this story -- not a summary pointing to the tech design, but the actual detail the implementer needs. The engineer should understand what modules they're working in, how data flows through this story's scope, and what constraints apply, without opening the tech design.

Select the relevant tech design sections (system context, module architecture, sequence/flow narratives) and include them at the scale they exist in the tech design, scoped to this story. If the tech design has a detailed flow for this story's capability, include that flow. If the tech design names specific modules with responsibilities, include those modules and responsibilities.

```markdown
## Technical Implementation

### Architecture Context

This story covers the session creation route and Convex persistence layer.

**Modules and Responsibilities:**

| Module | Responsibility | AC Coverage |
|--------|---------------|-------------|
| `src/routes/sessions.ts` | Route handler for POST /sessions. Validates request body against CreateSessionSchema, delegates to SessionService, returns 201 with session ID or appropriate error response. | AC-1.1, AC-1.2 |
| `src/services/session.service.ts` | Orchestration layer. Receives validated params, generates session ID (CUID2), calls ConvexClient to persist, returns Session object. | AC-1.1, AC-1.3 |
| `src/clients/convex.client.ts` | Persistence adapter. Inserts SessionRecord into Convex `sessions` table. Wraps Convex errors in SessionError for consistent error handling upstream. | AC-1.1 |

**Session Creation Flow:**

```
POST /sessions (body: { title, model })
    → Route validates body against CreateSessionSchema
    → SessionService.create({ title, model, userId })
        → Generate session ID (CUID2)
        → ConvexClient.insertSession({ id, title, model, userId, createdAt })
        → Return Session { id, title, model, createdAt }
    → Route returns 201 { sessionId: string }
```

On ConvexClient failure, SessionService throws SessionError (cause preserved). Route handler catches SessionError and returns 503 with `{ error: "session_creation_failed" }`. No retry in the request path -- fail fast.
```

### Interfaces & Contracts

The specific interfaces from the tech design that this story implements or consumes, at the same level of detail the tech design provides. Not all interfaces -- just the ones relevant to this story's ACs. Include type definitions, function signatures, and contract details so the engineer knows exactly what to implement without opening the tech design's Low Altitude section.

```markdown
### Interfaces & Contracts

**Creates:**

```typescript
// src/services/session.service.ts
interface SessionService {
  create(params: CreateSessionParams): Promise<Session>;
}

// src/routes/sessions.ts
// POST /sessions
// Request: { title: string; model: ModelId }
// Success: 201 { sessionId: string }
// Error: 400 { error: "invalid_params", details: ZodError }
// Error: 503 { error: "session_creation_failed" }
```

**Consumes:**

```typescript
// src/clients/convex.client.ts (from Story 0)
interface ConvexClient {
  insertSession(session: SessionRecord): Promise<Id<"sessions">>;
}
```

**Types (from Story 0):**

```typescript
interface CreateSessionParams {
  title: string;
  model: ModelId;
  userId: string;
}

interface Session {
  id: string;       // CUID2
  title: string;
  model: ModelId;
  createdAt: Date;
}

interface SessionRecord extends CreateSessionParams {
  id: string;
  createdAt: number; // Unix ms (Convex storage format)
}
```

**Validation Schema:**

```typescript
const CreateSessionSchema = z.object({
  title: z.string().min(1).max(200),
  model: z.enum(["gpt-4", "claude-3", "gemini-pro"]),
});
```
```

### TC to Test Mapping

The critical traceability link. Every TC from this story's functional section maps to a test approach. This section is derived from the tech design's TC-to-test mapping and test plan, filtered to this story's scope.

**No test-plan guessing.** If the tech design specifies test files, test case names, or test approaches for this story's scope, those exact details must appear here. The implementer should never need to derive test approaches that the tech design already decided and the team already reviewed.

```markdown
### TC -> Test Mapping

| TC | Test File | Test Description | Approach |
|----|-----------|------------------|----------|
| TC-1.1a | tests/service/sessions.test.ts | creates session with valid params | Service mock: mock ConvexClient.insertSession, call POST /sessions with valid body, assert 201 + sessionId in response |
| TC-1.1b | tests/service/sessions.test.ts | rejects missing title | Service mock: POST /sessions with `{ model: "gpt-4" }`, assert 400 + ZodError details |
| TC-1.1c | tests/service/sessions.test.ts | rejects invalid model | Service mock: POST /sessions with `{ title: "test", model: "invalid" }`, assert 400 |
| TC-1.2a | tests/service/sessions.test.ts | returns session ID in response body | Service mock: verify response body matches `{ sessionId: string }` |
| TC-1.3a | tests/service/sessions.test.ts | persists session to Convex | Service mock: verify ConvexClient.insertSession called with correct SessionRecord shape |
```

Every TC must appear in this table. If a TC has no test mapping, either the TC is untestable (return to BA/SM) or the tech design is missing a test boundary (return to Tech Lead self-review).

### Non-TC Decided Tests

Tests decided in the tech design that aren't 1:1 with a TC but fall within this story's scope. These are edge cases, collision tests, integration tests, or defensive tests that the tech design explicitly planned. They must be carried forward so the implementer knows they exist.

```markdown
### Non-TC Decided Tests

| Test File | Test Description | Source |
|-----------|------------------|--------|
| tests/service/sessions.test.ts | ConvexClient timeout returns 503, does not retry | Tech Design §3.2 Error Handling |
| tests/service/sessions.test.ts | CUID2 generation produces valid format | Tech Design §3.1 Session ID Strategy |
```

If the tech design has no non-TC tests for this story's scope, state explicitly: "None. Tech Design sections §X and §Y reviewed -- no additional tests beyond TC mappings."

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

**Must cite which tech design sections were checked.** An empty deviation with section references means "I checked these sections and this story aligns." An empty deviation with no references means "I didn't check." The citations use stable heading names, not line numbers.

```markdown
### Spec Deviation

None. Checked against Tech Design: §3.1 Session Creation Flow, §3.2 Error Handling, §Low Altitude — SessionService Interface.
```

Or when deviations exist:

```markdown
### Spec Deviation

Checked against Tech Design: §3.1 Session Creation Flow, §3.2 Error Handling, §3.4 Caching Strategy, §Low Altitude — SessionService Interface.

Deviations:
- Tech Design §3.4 specifies Redis caching for session lookups. Deferred to Story 3
  per dependency sequencing. Session reads in this story go directly to Convex.
- Tech Design §3.1 Flow 1 shows auth middleware. Auth is not in scope for Story 1;
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

Every story that completes technical enrichment must satisfy these non-negotiable requirements. These are the quality gate for this phase.

**1. Tech design shard present.** The Architecture Context and Interfaces & Contracts sections contain a substantial, story-scoped shard of the tech design -- not a summary or abstract. The implementer should not need to open the full tech design to implement this story.

**2. TC to test mapping present.** Every TC in the story's functional section has a corresponding entry in the TC-to-test mapping table, with test file, description, and approach carried forward from the tech design. Exception: Story 0 has no TCs (it's foundation setup), so it has no TC mapping. Story 0 gets simplified technical sections instead (see below).

**3. Non-TC decided tests present.** Any tests decided in the tech design that fall within this story's scope but aren't 1:1 with a TC are listed in the Non-TC Decided Tests section. If none exist, the section explicitly states "None" with checked tech design section references.

**4. Technical DoD present.** The technical checklist includes specific verification commands and regression expectations. Not "tests pass" -- which tests, which commands, what counts as passing.

**5. Spec deviation field present with section citations.** Every story has a spec deviation field, even when empty, citing which tech design sections were checked. An empty field with citations means "I checked and there are no deviations." A missing field or missing citations means "I didn't check."

**6. Technical sections describe targets, not steps.** The Architecture Context names modules and flows. The Interfaces section names contracts. Neither prescribes the implementation sequence. The engineer decides how to get there -- the story describes where "there" is. Substantial detail and targets-not-steps coexist: a story can include detailed flows, interfaces, and test plans without prescribing the implementation order.

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

The ultimate validation for technically enriched stories: **Could an engineer implement from this story alone, without reading the full tech design?**

This is not a rhetorical question. Apply it literally:

- Does the story tell the engineer which modules to create or modify, with enough flow/architecture detail to understand how they connect?
- Does the Interfaces & Contracts section carry the actual type definitions and signatures, not just names?
- Does the TC-to-test mapping tell them exactly what tests to write, with file names and approaches from the tech design?
- Are non-TC decided tests listed so the engineer doesn't have to discover them independently?
- Does the technical DoD tell them how to verify their work?
- Does the spec deviation field tell them where this story intentionally diverges from the tech design?
- Are there ambiguities that would force the engineer to choose between interpretations?

The tech design is available as fallback -- for rationale, alternatives-considered, or resolving ambiguity. But the story should be sufficient for implementation without it. If the engineer would need to read the full tech design to understand what to build, the tech design shard is too thin. If the engineer would need to derive test approaches from scratch, the TC-to-test mapping is incomplete.

The consumer gate is the quality bar. Stories that pass it are ready for implementation.

---

## Coherence Check Against Tech Design

After enriching all stories, verify completeness against the tech design:

**Interface coverage:** Every interface defined in the tech design's Low Altitude section should appear in at least one story's Interfaces & Contracts section. An interface with no story assignment is either dead code in the design or a gap in the story breakdown.

**Module coverage:** Every module from the tech design's Module Responsibility Matrix should appear in at least one story's Architecture Context. A module with no story is either unnecessary or missing from the sharding.

**Test mapping coverage:** The union of all stories' TC-to-test mapping tables should cover every TC from the epic. Compare against the coverage gate table from the functional story sharding phase. Additionally, every non-TC test decided in the tech design should appear in at least one story's Non-TC Decided Tests section.

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

**Over-compressed tech sections.** The Architecture Context is 10 lines summarizing 300 lines of relevant tech design content. The engineer can't write the decided tests or hit the right interfaces without opening the tech design. The enricher compressed instead of curated. The fix: go back to the tech design, identify the sections relevant to this story, and include them at the scale they exist -- edited for story scope, not condensed for brevity.

**Missing non-TC decided tests.** The tech design explicitly planned edge case tests, collision tests, or integration tests for this story's scope, but the story's technical section doesn't mention them. The implementer either misses them entirely or has to discover them by reading the tech design -- which defeats the purpose of enrichment.

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
- Architecture Context (story-scoped tech design shard)
- Interfaces & Contracts (full type definitions and signatures)
- TC to Test Mapping (every TC with test file, description, approach)
- Non-TC Decided Tests (tech-design-decided tests beyond TC mappings)
- Risks & Constraints
- Spec Deviation (with checked section citations)
- Technical Checklist

These complete stories are the sole implementation artifact. Engineers receive them, form an implementation plan, and execute. No prompt packs, no orchestration scripts, no intermediate artifacts between the story and the code.

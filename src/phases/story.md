# Story Sharding

**Purpose:** Break an epic into functional stories -- discrete, independently acceptable vertical slices of user-facing value.

Stories are where product management meets project management. They group acceptance criteria from the epic into deliverable units of work, sequence them based on dependencies, and carry enough functional detail that a PO can accept from the story alone. The story is the acceptance artifact. The engineer receives it, implements from it, and the PO accepts against it.

**You create the functional stories. You don't add technical implementation detail.** Technical sections (implementation targets, test mapping, verification criteria) are added by the Tech Lead in the Story Technical Enrichment phase that follows.

### Special Artifacts

- **Story 0:** Foundation setup -- types, fixtures, error classes, project config. Minimal or no TDD cycle.
- **Feature 0:** Stack standup -- auth, connectivity, integrated skeleton with no product functionality. Used when building on a new stack where the foundation doesn't exist yet.

## Validator Role

Before creating stories, validate the Tech Design:

- Are interfaces clear enough to identify logical groupings of ACs?
- Is the TC-to-test mapping complete?
- Can you identify logical groupings of ACs that form coherent units of work?
- Can you trace dependency chains between groups?

If issues found, return to Tech Lead for revision. Don't shard from a broken design.

---

## What a Story Is

A story is a functional scoping and acceptance artifact. It spans two concerns:

**Product management:** The story carries enough functional detail -- full acceptance criteria with test conditions -- that work can be accepted from the story alone without referencing back to the epic. A PO reads the story's ACs and TCs to determine whether the delivered work meets the requirement.

**Project management:** The story groups ACs from the epic into a deliverable unit, sequences it relative to other stories, and defines what "done" looks like. Sometimes you'd shard differently for a two-person team than for a solo dev. The story is where that project-level adaptation happens.

## What a Story Is NOT

A story at this phase is not an implementation reference. It does not carry file lists, interface definitions, or implementation patterns -- those are added by the Tech Lead in the Story Technical Enrichment phase, which shards the tech design into each story's technical section. The functional story provides light technical context as orientation ("this story involves the session creation route and Convex persistence layer") sufficient for the Tech Lead to plan enrichment.

After technical enrichment, the story becomes the sole implementation artifact -- self-contained with substantial tech design content, full test mappings, and interface definitions. Engineers implement from complete stories using their own judgment and plan mode. The tech design becomes optional fallback for rationale and cross-cutting context.

---

## Story 0: Foundation Story

Every feature starts with Story 0. It establishes the shared foundation that all subsequent stories build on.

### What Story 0 Contains

- **Type definitions** -- All interfaces from the tech design's Low Altitude section
- **`NotImplementedError` class** -- Custom error for stubs (if not already in the codebase)
- **Error classes** -- Feature-specific errors defined in the tech design
- **Test fixtures** -- Mock data matching the data contracts in the epic
- **Test utilities** -- Shared helpers for test setup (factory functions, mock builders)
- **Project config** -- Test config, path aliases, setup files, environment config

### Pragmatic Additions

The default for Story 0 is foundation-only with no TDD cycle -- types and fixtures don't need test-driven development. However, projects may pragmatically include foundational endpoints or smoke tests that verify the infrastructure actually works. A health check endpoint that proves the server starts and dependencies connect is a reasonable Story 0 addition. The key constraint is that Story 0 establishes what all subsequent stories depend on -- it's the shared foundation, not a feature delivery.

Use judgment. If something needs TDD, it probably belongs in Story 1. If it's verifying that the infrastructure from Story 0 is wired correctly, it can live in Story 0 with a simplified structure.

Use these defaults for smoke tests:
- Include smoke tests in Story 0 when they validate shared wiring (boot, connectivity, required env/config).
- Skip smoke tests in Story 0 when they would validate feature behavior better covered by Story 1+ TDD.
- Keep Story 0 smoke coverage minimal; this is a foundation check, not feature verification.

### Exit Criteria

- [ ] All type definitions from tech design created
- [ ] Test fixtures match data contracts
- [ ] Error classes and `NotImplementedError` available
- [ ] TypeScript compiles clean
- [ ] Project config validated (env vars, test runner, linter)

---

## Story Derivation

### Stories as Project-Level Adaptation

Stories are a project-level adaptation of the epic. They group acceptance criteria into implementable units of work based on:

- **Functional coherence** -- ACs that belong together because they describe a single user capability
- **Dependency sequencing** -- What must exist before this work can begin
- **Scope manageability** -- Enough work to be meaningful, not so much that it's unwieldy
- **Project pragmatics** -- Team size, parallelization opportunities, risk isolation

The tech design's chunk breakdown may inform this grouping -- chunks often align with natural story boundaries because both are reasoning about logical units of work. But the story doesn't mechanically derive from chunks. Sometimes a chunk splits across stories, or multiple chunks merge into one story, or the sequencing differs from the tech design's ordering.

### Story Types

**Story 0: Foundation**
- Establishes shared infrastructure
- Minimal or no TDD cycle
- No feature delivery (or minimal foundational verification)

**Story 1-N: Feature Stories**
- Deliver user-facing functionality
- Full TDD cycle: Skeleton, Red, Green, Gorilla, Verify
- Each story is independently verifiable and acceptable

---

## Story Structure (Functional Sections)

This template covers the functional half of a complete story. The Tech Lead adds the technical half during Story Technical Enrichment.

```markdown
# Story N: [Title]

## Objective

What this story delivers. What a user/consumer can do after this story ships that they couldn't before.

## Scope

### In Scope
- Specific capabilities this story delivers
- Which endpoints, features, or behaviors

### Out of Scope
- What's explicitly excluded (and which story handles it)

## Dependencies / Prerequisites

- Story N-1 must be complete
- Specific capabilities that must exist (not file lists -- functional prerequisites)

## Acceptance Criteria

**AC-X.Y:** [Acceptance criterion title]

- **TC-X.Ya: [Test condition name]**
  - Given: [precondition]
  - When: [action]
  - Then: [expected outcome]
- **TC-X.Yb: [Test condition name]**
  - Given: [precondition]
  - When: [action]
  - Then: [expected outcome]

**AC-X.Z:** [Next acceptance criterion]
[... full Given/When/Then for each TC]

## Error Paths

| Scenario | Expected Response |
|----------|------------------|
| [Error condition] | [HTTP status and error code] |

## Definition of Done

- [ ] All ACs met
- [ ] All TC conditions verified
- [ ] PO accepts
```

**Ownership boundary:** The BA/SM authors these functional sections. The PO accepts from them. Technical implementation sections are added below this boundary by the Tech Lead in the next phase.

---

## Why Full AC/TC Detail in Stories

The story is the acceptance artifact. If TCs are reduced to one-line summaries ("AC-2.5: Duplicate entries rejected"), a PO can't accept from the story alone -- they need the epic open to see what "rejected" actually means. Full Given/When/Then detail makes the story self-contained for acceptance.

Stories may refine or add specificity to the epic's TCs. A story TC can be more implementation-aware than the epic's version ("Given: A session exists; request contains one `user-message` entry and one `assistant-message` entry") because the story is closer to the work. The epic's TC is the source; the story's TC is the acceptance-ready version for this scope.

---

## Integration Path Trace (Required)

After defining all stories and before handing off to the Tech Lead, trace each critical end-to-end user path through the story breakdown. This catches cross-story integration gaps that per-story AC/TC coverage cannot detect.

Per-story validation checks whether each story is internally complete -- ACs covered, scope defined, prerequisites met. It does not check whether the *union* of all stories produces a connected system. A relay module, a bridge between subsystems, a glue handler that routes messages -- these can fall through the cracks when each story takes one side of the boundary and no story owns the seam.

### How to Trace

1. List the 1-3 most important user paths (from the epic's flows)
2. Break each path into segments (each arrow in the sequence diagram)
3. For each segment, identify which story owns it
4. Verify at least one TC in that story exercises the segment

Any segment with no story owner is an integration gap. Fix before handing off.

### Format

| Path Segment | Description | Owning Story | Relevant TC |
|---|---|---|---|
| Client -> POST /sessions | Create session | Story 1 | TC-1.1a |
| Client -> POST /sessions/:id/turns | Ingest turn | Story 2 | TC-2.1a |
| Ingestion -> Redis DEL | Cache invalidation | Story 2 (impl) / Story 3 (verification) | TC-3.2d |
| Client -> GET /sessions/:id/history | Retrieve history | Story 3 | TC-3.1a |
| History -> Redis check | Cache hit | Story 3 | TC-3.2b |
| History -> Convex fallback | Cache miss | Story 3 | TC-3.2a |
| WS -> portlet | Shell relays to iframe | ??? | ??? |

Empty cells ("???") are integration gaps. They block handoff.

### Why Per-Story Checks Don't Catch This

Story-level validation asks: "Are this story's ACs covered?" These are within-story completeness checks. The Integration Path Trace is a cross-story coverage check -- does every segment of the critical user path have a story owner? A tech design can perfectly describe how components interact while no story actually owns implementing the glue between them.

---

## Coverage Gate

Before proceeding to Story Technical Enrichment, verify that every AC and TC from the epic is assigned to exactly one story. Build a coverage table:

| AC | TC | Story | Notes |
|----|-----|-------|-------|
| AC-1.1 | TC-1.1a | Story 1 | |
| AC-1.1 | TC-1.1b | Story 1 | |
| AC-1.2 | TC-1.2a | Story 1 | |
| AC-2.1 | TC-2.1a | Story 2 | |
| ... | ... | ... | |

**Rules:**
- Every AC must appear at least once
- Every TC must appear exactly once (no duplication across stories, no gaps)
- Unmapped TCs block handoff -- they indicate a story is missing scope or the epic has orphaned requirements

---

## Validation Before Handoff

Before handing functional stories to the Tech Lead for technical enrichment:

- [ ] Every AC from the epic is assigned to a story
- [ ] Every TC from the epic is assigned to exactly one story
- [ ] Stories sequence logically (read before write, foundation before features)
- [ ] Each story has full Given/When/Then detail for all TCs
- [ ] Integration path trace complete with no gaps
- [ ] Coverage gate table complete
- [ ] Error paths documented per story
- [ ] Story 0 covers types, fixtures, error classes, project config
- [ ] Each feature story is independently acceptable by a PO

**Self-review (CRITICAL):**
- Read each story fresh, as if someone else wrote it
- Can you explain why each AC belongs in this story?
- Does each story tell a coherent "what the user can do after" narrative?

**Downstream consumer:** The Tech Lead validates functional stories by confirming they can add technical sections. If they can't identify implementation targets for a story's ACs, the story isn't ready.

---

## Output: Functional Stories

For a typical feature, you produce:
- Story 0: Foundation setup (types, fixtures, error classes)
- Stories 1-N: Feature stories with full acceptance criteria, error paths, and definition of done
- Integration path trace table
- Coverage gate table

These are functional stories -- the acceptance half. The Tech Lead adds technical sections (implementation targets, TC-to-test mapping, technical DoD, spec deviation field) in the Story Technical Enrichment phase before stories go to engineers.

---

## Iteration is Expected

- Tech Lead finding functional gaps -> iterate
- Coverage table revealing orphaned TCs -> iterate
- Integration trace exposing seam gaps -> iterate
- Multiple rounds is normal, not failure

Don't expect one-shot perfection. The structure supports iteration.

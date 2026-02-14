# Story Sharding & Orchestration

**Purpose:** Parse feature into stories and generate prompt packs for execution.

**You create the work. You don't execute it.** Stories and prompts go to Senior Engineer agents for implementation.

**The Orchestrator runs this phase and continues into Phase 5 (Execution).** The same orchestrating context that shards stories also coordinates their execution — launching Senior Engineer sessions, managing validation, and driving the pipeline.

### Special Artifacts

- **Story 0:** Infrastructure setup — types, fixtures, error classes, stubs. No TDD cycle.
- **Feature 0:** Stack standup — auth, connectivity, integrated skeleton with no product functionality. Used when building on a new stack where the foundation doesn't exist yet.

## Dual Role: Validator and Orchestrator

### As Validator

Before creating stories, validate the Tech Design:
- Can you map design chunks to discrete stories?
- Are interfaces clear enough to write implementation prompts?
- Is test mapping complete?

If issues found → return to Tech Lead for revision.

### As Orchestrator

Once validated, produce:
- Story breakdown
- Prompt packs for each story
- Orchestration of Senior Engineer sessions

---

## Story 0: Infrastructure Story

Every feature starts with Story 0. It establishes the shared foundation that all subsequent stories build on. Story 0 has no user-facing functionality and no TDD cycle — it's pure setup.

### What Story 0 Contains

- **`NotImplementedError` class** — Custom error for stubs (if not already in the codebase)
- **Type definitions** — All interfaces from the tech design's Low Altitude section
- **Test fixtures** — Mock data matching the data contracts in the feature spec
- **Test utilities** — Shared helpers for test setup (factory functions, mock builders)
- **Error classes** — Feature-specific errors defined in the tech design
- **Project config** — Any test config, path aliases, or setup files needed

### What Story 0 Does NOT Contain

- No tests (types and fixtures don't need TDD)
- No implementation logic
- No component/hook/API stubs (those come in Story 1+ skeleton phase)
- No user-facing functionality

### Why It's Always First

Story 0 establishes the contracts that all subsequent stories depend on. Types defined here become the interfaces that stubs implement, tests assert against, and implementations fulfill. If types change mid-feature, every downstream story is affected — getting them right first reduces rework.

### Story 0 Prompt Structure

Story 0 uses a simplified prompt structure since there's no TDD cycle:

```
story-0-infrastructure/
├── story.md
├── prompt-0.1-setup.md      # Creates all infrastructure
└── prompt-0.R-verify.md     # Verifies setup complete (typecheck passes)
```

### Exit Criteria

- [ ] All type definitions from tech design created
- [ ] Test fixtures match data contracts
- [ ] `NotImplementedError` class available
- [ ] TypeScript compiles clean
- [ ] No tests (none expected)

---

## Story Derivation

### From Tech Design Chunks to Stories

Tech designs include "chunks" — logical groupings of work:

```markdown
## Chunk 1: Initial Load
**Scope:** Page mount, data fetching, loading/error states
**ACs:** AC-1 to AC-9
**Phases:** Skeleton + Red, Green
```

Each chunk maps to a story, usually 1:1. Sometimes a large chunk splits into multiple stories (if it has too many TCs for one execution cycle) or small related chunks merge into one story. Use judgment — the goal is stories that are independently executable and verifiable.

### Story Types

**Story 1-N: Feature Stories**
- Deliver user-facing functionality
- Full TDD cycle: Skeleton → Red → Green → Gorilla → Verify

### Story Structure

```markdown
# Story N: [Title]

## Overview
What this story delivers. What user can do after.

## Prerequisites
- Story N-1 must be complete
- These files exist: [list]

## ACs Covered
- AC-X: [summary]
- AC-Y: [summary]

## Files

**New:**
- src/path/to/NewFile.tsx

**Modified:**
- src/path/to/ExistingFile.tsx (add X functionality)

## Test Breakdown
- NewFile.test.tsx: 8 tests (TC-X to TC-Y)
- ExistingFile.test.tsx: 4 additional tests (TC-Z)
- **Story total:** 12 tests
- **Running total:** 24 tests (12 from Story 1 + 12 new)

## Prompts
| Phase | File | Purpose |
|-------|------|---------|
| Skeleton+Red | prompt-N.1-skeleton-red.md | Stubs + tests |
| Green | prompt-N.2-green.md | Implementation |
| Verify | prompt-N.R-verify.md | Verification checklist |
```

---

## Prompt Pack Creation

Each prompt must be **self-contained**. A fresh agent executes without conversation history.

### Prompt Structure

```markdown
# Prompt N.X: [Phase Name]

## Context
You are implementing Story N of [Feature].

**Working Directory:** /absolute/path/to/project

**Prerequisites complete:**
- Story N-1 files exist and tests pass
- [List specific files]

## Reference Documents
- Tech Design: path/to/tech-design.md (section X)
- Feature Spec: path/to/feature.md (ACs X-Y)

## Task

### Files to Create/Modify
1. `src/path/to/File.tsx`
   - Purpose: [what it does]
   - [Key requirements]

### Code
[For skeleton/green: include actual code, not "similar to before"]

```typescript
// Full implementation here
```

## Constraints
- Do NOT implement beyond this story's scope
- Do NOT modify files not listed
- Use exact type names from tech design
- Use exact data-testid values specified

## Verification
```bash
npm test -- --testPathPattern="Feature"
```
Expected: [X tests PASS, Y tests ERROR]

## Done When
- [ ] All files created/modified
- [ ] Test state matches expected
- [ ] TypeScript compiles
```

### Key Point: Inline Content

Tech design is referenced but **content should be IN the prompt**. Don't require the model to go read another doc.

The prompt pack is self-contained. Reference documents are for human traceability, not model execution.

### Prompt Anti-Patterns

| ❌ Bad | ✅ Better |
|--------|----------|
| "Create the component similar to what we discussed" | Full code in prompt |
| "Now implement the hook" | Complete context + code |
| "Make sure it works correctly" | Explicit verification commands + expected output |
| "See the tech design for details" | Details inlined in prompt |

---

## Prompt Validation

Before giving prompts to Senior Engineer, validate them. At minimum: self-review for completeness, then have a fresh agent confirm they can execute from the prompt alone.

For the full validation pattern — dual-validator, parallel validation, fix cycles, and consolidation — see `references/execution-orchestration.md`.

---

## Orchestration

### Launching Senior Engineer Sessions

For each story:
1. Provide Skeleton+Red prompt → Fresh session executes
2. Verify Red state (tests ERROR)
3. Provide Green prompt → Fresh session executes
4. Verify Green state (tests PASS)
5. Human does Gorilla testing
6. Provide Verify prompt → Fresh session validates
7. Story complete

### Handling Issues

**Story blocked:** Document blocker, skip to unblocked work if possible, escalate to human.

**Ground-level discovery:** Senior Engineer found spec gap → document, get human approval, update spec if needed.

**Test count mismatch:** Investigate before proceeding. Running totals must be accurate.

---

## Running Test Totals

Track cumulative counts across stories:

```
Story 0 (infrastructure): 0 tests (setup only)
Story 1: 12 tests
Story 2: 12 + 15 = 27 tests
Story 3: 27 + 10 = 37 tests
```

**Previous stories' tests must keep passing.** If Story 2 breaks Story 1 tests, that's a regression to fix before proceeding.

---

## Impl vs Verifier Prompts

The same reference info, the same context, but different role and focus.

| Prompt Type | Focus | Lens |
|-------------|-------|------|
| Implementation | Create/modify code | Builder |
| Verification | Check against spec | Auditor |

Same material, different lens. The verifier prompt should be thorough — that's the point.

---

## Iteration is Expected

- Engineer finding own issues → iterate
- Verifier finding issues → iterate
- Multiple rounds is normal, not failure

Don't expect one-shot perfection. The structure supports iteration.

---

## Output: Stories + Prompt Packs

For a typical feature, you produce:
- Story 0: Infrastructure setup
- Stories 1-N: Feature stories with full TDD cycle
- Each story has: overview (story.md), skeleton-red prompt, green prompt, verify prompt

Each prompt pack is self-contained. Senior Engineers execute with zero prior context.

---

# Story Prompts Reference

Stories are the execution layer that translates specs and designs into working code.

## The Confidence Chain

```
AC (requirement) → TC (testable condition) → Test (code) → Implementation → Verification
```

## Story Types

### Infrastructure Story (Story 0)

Sets up shared infrastructure before feature stories begin.

```
story-0-infrastructure/
├── story.md
├── prompt-0.1-setup.md      # Creates all infrastructure
└── prompt-0.R-verify.md     # Verifies setup complete
```

**Deliverables:** NotImplementedError, types, fixtures, test utilities.

### Feature Story (Story 1+)

Delivers user-facing functionality with TDD.

```
story-N-{description}/
├── story.md
├── prompt-N.1-skeleton-red.md   # Stubs + tests
├── prompt-N.2-green.md          # Implementation
└── prompt-N.R-verify.md         # Verification
```

---

## Deriving Stories from Tech Design

1. **Identify the chunk** in tech design
2. **List the TCs** from feature spec — include TC IDs explicitly (e.g., TC-6a, TC-6b)
3. **Pull test mapping** from tech design's TC-to-test table
4. **Identify files** to create/modify

**TC traceability carries through:** The story should list which TCs it covers. Tests written for this story should reference those TC IDs in comments or test names.

---

## Writing Self-Contained Prompts

Phase prompts must be **self-contained**. A fresh agent with no conversation context should be able to execute.

### Prompt Structure (Composable Prompt Pack)

```markdown
# Prompt N.X: [Phase Name]

## Context
Product summary
  → Project summary
    → Feature summary (referenced, key points inlined)
      → Story summary (what's in scope here)
        → Task details (what's done here)

**Working Directory:** /absolute/path/to/project

**Prerequisites complete:**
- Story N-1 files exist and tests pass
- [List specific files]

## Reference Documents
(For human traceability — key content inlined below)
- Tech Design: path/to/tech-design.md (section X)
- Feature Spec: path/to/feature.md (ACs X-Y)

## Task

### Files to Create/Modify
1. `src/path/File.tsx` — [Purpose, requirements]

### Implementation Requirements
- [Specific requirement 1]
- [Specific requirement 2]

### Code
[For skeleton/green: include actual code]

## Constraints
- Do NOT implement beyond this phase's scope
- Do NOT modify files outside the specified list
- Use exact type names from tech design
- Use exact data-testid values specified

## If Blocked or Uncertain
- If you encounter inconsistencies between the prompt, tech design, or feature spec — **stop and ask** before proceeding
- If something doesn't line up (signatures don't match, test counts conflict, a dependency is missing) — surface it rather than silently resolving it
- If you are blocked (missing file, failing prerequisite, unclear requirement) — document what you attempted, what's not working, and what you think the resolution is, then return to the orchestrator
- Do NOT work around ambiguity or inconsistencies without approval

## Verification
When complete:
1. Run: `npm test -- --testPathPattern="FeaturePage"`
2. Expected: [X tests pass | X tests error]
3. Run: `npx tsc --noEmit`
4. Expected: No errors

## Done When
- [ ] All files created/modified
- [ ] Test state matches expected
- [ ] TypeScript compiles
```

### Key Point: Content IN the Prompt

Tech design is referenced but content should be IN the prompt. Don't require model to go read another doc.

Reference files are for human traceability. The model executes from what's inlined.

---

## Prompt Anti-Patterns

| ❌ Bad | ✅ Better |
|--------|----------|
| "Create the component similar to before" | "Create FeatureList.tsx with: [full code]" |
| "Now implement the hook" | "Implement useFeatureData. Previous phases created types in..." |
| "Make sure it works" | "After implementation, `npm test` shows 8 tests passing" |
| "See tech design for interface" | Interface definition inlined in prompt |

---

## Story Derivation Example

**From tech design chunk:**
```markdown
## Chunk 2: Select and Return
**Scope:** Filtering, selection, return data
**ACs:** AC-17 to AC-26
```

**Becomes:**
```markdown
## Story 2: Select and Return

### TCs Covered
TC-17 to TC-27 (from feature spec)

### Files
- New: `src/hooks/useLocationSelection.ts`
- Modified: `src/components/LocationList.tsx`

### Test Breakdown
- useLocationSelection.test.ts: 6 tests (NEW)
- LocationList.test.tsx: 10 additional tests (MODIFY)
- **Story total:** 16 tests
- **Running total:** 26 + 16 = 42 tests
```

---

## Impl vs Verifier Prompts

Same reference info, same context. Different role and focus.

| Prompt Type | Focus | Lens |
|-------------|-------|------|
| Implementation | Create/modify code | Builder |
| Verification | Check against spec | Auditor |

The verifier prompt should use a thorough, detail-oriented model (GPT 5x, Codex). The point is to catch what builders miss.

---

## Key Principles

1. **Self-contained** — No reliance on conversation history
2. **Explicit** — Exact file paths, exact code, exact test counts
3. **Verifiable** — Clear pass/fail criteria
4. **Traceable** — TC numbers in comments link back to spec
5. **Inlined** — Key content in prompt, not just references

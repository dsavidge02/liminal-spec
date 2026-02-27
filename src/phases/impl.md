# Implementation

**Purpose:** Implement from complete stories using TDD discipline and engineering judgment.

You receive a story with functional sections (ACs, TCs, error paths, functional DoD) and technical sections (architecture context, interfaces, TC-to-test mapping, technical checklist). That story, plus the epic and tech design as reference artifacts, is your implementation context. No prompt packs, no orchestration scripts -- you plan and execute from the story.

---

## The Implementation Mindset

You're an engineer, not a script runner. The story tells you *what* to build and *how to verify it*. You decide *how* to build it. Use plan mode to form your approach, get approval, then execute.

The story's technical sections describe targets -- modules to create, interfaces to implement, tests to write. They don't prescribe the sequence. You read the story, understand the acceptance criteria, review the technical sections, and build an implementation plan that makes sense for the specific codebase and constraints you're working in.

---

## Working from a Story

### What You Receive

A complete story with two halves:

**Functional (from BA/SM):**
- Objective -- what the user can do after
- Scope -- what's in and out
- Acceptance criteria with full Given/When/Then test conditions
- Error paths
- Functional Definition of Done

**Technical (from Tech Lead):**
- Architecture Context -- which modules, which flows
- Interfaces & Contracts -- what to create, what to consume
- TC to Test Mapping -- which test file, which approach for each TC
- Risks & Constraints -- what to watch out for
- Spec Deviation -- where this story intentionally differs from the tech design
- Technical Checklist -- specific verification commands

**Reference artifacts** (in the project repo, consult when needed):
- **Epic** -- full functional requirements, data contracts, user flows
- **Tech Design** -- architecture rationale, alternatives-considered, cross-cutting context

The story is your primary and usually sufficient guide. The technical sections contain substantial tech design content sharded into this story's scope -- architecture, interfaces, flows, and test plans. Consult the epic or tech design only when the story is ambiguous, when you need rationale for a decision, or when something seems wrong. If you find a mismatch between the story and the tech design, stop and flag it for reconciliation before proceeding.

### Plan Before You Build

Before writing code, form an implementation plan:

1. Read the story's functional and technical sections -- this should give you everything needed
2. Identify the implementation sequence (which modules in what order)
3. Note any risks or ambiguities from the story's Risks & Constraints
4. Check the spec deviation field -- know where this story diverges from the design
5. If anything is unclear or seems incomplete, consult the epic or tech design for context

If using plan mode, present your plan for approval before executing. The story is the planning input -- it should give you everything you need to form a concrete plan. If you find yourself needing to read the full tech design to understand what to build, flag this as a potential enrichment gap.

---

## The TDD Cycle (Recommended)

TDD is the recommended implementation approach. The story's TC-to-test mapping provides the test specification; the TDD cycle provides the implementation discipline. The verification gate (technical checklist) is the hard requirement -- the process is the engineer's judgment.

Each story follows: **Skeleton -> TDD Red -> TDD Green -> Self-Review -> Gorilla -> Verify**.

### Skeleton Phase

**Purpose:** Create structural scaffolding. Validate architecture before writing logic.

**Deliverables:**
- File structure (all directories and files)
- Type definitions (if not in Story 0)
- Component/function stubs that throw `NotImplementedError`
- Export statements
- Route registration (wired to stubs)

**The NotImplementedError Pattern:**

```typescript
// errors.ts
export class NotImplementedError extends Error {
  constructor(message = 'Not implemented') {
    super(message);
    this.name = 'NotImplementedError';
  }
}
```

**Stub Examples:**

```typescript
// Component
import { NotImplementedError } from '@/errors';
import type { LocationListProps } from '@/types';

export function LocationList(props: LocationListProps): JSX.Element {
  throw new NotImplementedError('LocationList');
}

// Hook
import { NotImplementedError } from '@/errors';
import type { UseLocationsReturn } from '@/types';

export function useLocations(): UseLocationsReturn {
  throw new NotImplementedError('useLocations');
}

// API function
import { NotImplementedError } from '@/errors';

export const locationApi = {
  getAll: async (sai: string) => {
    throw new NotImplementedError('locationApi.getAll');
  },
};
```

**Exit Criteria:**
- [ ] All files created per story's Architecture Context
- [ ] TypeScript compiles without errors
- [ ] All exports in place
- [ ] Stubs throw NotImplementedError
- [ ] No actual logic implemented

---

### TDD Red Phase

**Purpose:** Write tests that assert real behavior. Tests will ERROR because stubs throw.

**Critical Rule: Assert Behavior, Not Errors**

```typescript
// CORRECT - Asserts expected behavior
it('TC-10: renders location rows', () => {
  render(<LocationList locations={mockLocations} />);
  expect(screen.getByText('125 York Street')).toBeInTheDocument();
});

// WRONG - This passes before AND after implementation
it('TC-10: renders location rows', () => {
  expect(() => render(<LocationList locations={mockLocations} />))
    .toThrow(NotImplementedError);
});
```

**Why tests ERROR:** Stubs throw before assertions execute. The test is correctly written -- it just can't complete yet.

**Why the anti-pattern is dangerous:** Testing that NotImplementedError throws will PASS both before AND after implementation. You've verified nothing about actual behavior. This defeats the entire purpose of TDD.

Use the story's TC-to-test mapping table to know which tests to write, in which files, with which approach. The mapping is your test specification.

**Exit Criteria:**
- [ ] Tests written for all TCs in the story
- [ ] Tests assert behavior (not error throwing)
- [ ] Tests ERROR when run (stubs throw)
- [ ] Previous story tests still PASS
- [ ] Full lint/format/typecheck pipeline passes (everything except tests)
- [ ] **Commit checkpoint created** before proceeding to Green

The Red exit gate must run the full quality pipeline minus tests. Tests are expected to fail (stubs throw), but all other quality checks -- formatting, linting, type checking -- must pass. If the project defines a `red-verify` script (or equivalent), use it.

**Why this matters:** If Red produces lint-dirty code, Green implements against it. When lint failures surface later, fixing them risks destroying working Green implementation. Catch it at Red.

**Commit boundary:** Create a commit at Red completion before entering Green. This preserves a clean audit trail (requirement -> red tests -> implementation) and provides a rollback point if Green goes sideways.

---

### TDD Green Phase

**Purpose:** Implement to make tests pass. Replace stubs with real logic.

**Implementation Order:**
Start with leaf dependencies, move up:
1. API layer (no dependencies)
2. Hooks/services (depend on API)
3. Components/handlers (depend on hooks/services)

This lets you test each layer as you build.

**Red Test Immutability:**

Red tests are the behavioral contract for Green. Do not modify test files during Green.

Green implementation must satisfy the tests as written in Red. If a test file is modified during Green, that's a signal something is wrong -- either the Red tests were incorrect (fix in a separate cycle, not silently during Green) or the implementation is gaming the tests rather than satisfying intent.

If test files genuinely need editing during Green (e.g., environment setup fixes, not assertion changes), the change must be explicitly reviewed during verification. The verifier checks git history on affected test files to confirm the changes preserved AC/TC intent rather than weakening checks.

**Exit Criteria:**
- [ ] All tests PASS
- [ ] No NotImplementedError remaining in implemented code
- [ ] Implementation matches tech design interfaces
- [ ] No over-engineering beyond what tests require
- [ ] No test files modified (or modifications explicitly justified and reviewed)
- [ ] Full verification pipeline passes, including test immutability check

---

### Self-Review Practice

After completing Red and Green phases, review your own work before proceeding. This is a recommended engineering practice, not an orchestrated mandate.

**After Red:** Review your tests critically. Do they assert the right behavior? Do they cover the TCs from the story? Are mock setups realistic? Would these tests catch a broken implementation?

**After Green:** Review your implementation critically. Does it satisfy the tests for the right reasons? Did you introduce any shortcuts that technically pass tests but miss the intent? Are there edge cases the tests don't cover that you should flag?

Self-review catches issues cheaper than verification does. If you find something non-controversial, fix it. If you find something that requires judgment, document it and surface it.

---

### Gorilla Testing Phase

**Purpose:** Human-in-loop, ad hoc testing. Catches "feels wrong."

After TDD Green, before formal verification:
- Run the feature manually
- Try weird inputs
- Click things in unexpected order
- Use it like a real user would
- Look for anything that feels broken

Gorilla testing is not scripted, not automated, not systematic. The point is to catch things tests don't cover -- UX issues, edge cases you didn't think of, flows that technically work but feel wrong.

TDD ensures correctness against specified behavior. But specs can miss things. Tests can miss things. "Correct" isn't always "good." Gorilla testing legitimizes unstructured exploration within the structured process. It's not a failure of rigor -- it's a recognition that humans catch things automation doesn't.

**Exit Criteria:**
- [ ] Human has used the feature manually
- [ ] No "feels wrong" moments (or they're documented/fixed)
- [ ] Ready for formal verification

---

### Verify Phase

**Purpose:** Formal verification. Full test suite, types, lint.

Run the project's `verify` or `verify-all` script. If the project doesn't define these, run the components individually:

```bash
npm run format:check       # Formatting correct
npm run lint               # No lint errors
npm run typecheck          # No type errors
npm test                   # All tests pass
```

**Checklist Template:**

```markdown
## Story N Verification

### Automated
- [ ] All tests pass (this story + previous)
- [ ] TypeScript compiles clean
- [ ] Lint passes

### Manual (from Gorilla)
- [ ] Feature works as expected
- [ ] No console errors
- [ ] Previous functionality still works

### Implementation Details
- [ ] Uses correct types from tech design
- [ ] Matches interface contracts from story
- [ ] Spec deviations documented (per story's Spec Deviation field)
```

Run a shim/fudge audit as a required gate:
- Flag temporary shims/placeholders in non-test code presented as final behavior
- Flag internal-module mocking that bypasses real in-process integration
- If any shim is intentionally temporary, require explicit owner and removal condition

Report: PASS or FAIL with TC-by-TC status, plus shim audit status and findings.

**Exit Criteria:**
- [ ] All checklist items pass
- [ ] Story marked complete
- [ ] Ready for next story

---

## Handling Issues

### Inconsistencies or Ambiguity

If something doesn't line up -- the story's interfaces don't match the tech design, TCs conflict, a dependency seems wrong, or requirements contradict each other -- **stop and ask**. Do not silently resolve inconsistencies. The confidence chain depends on traceability, and a quiet workaround creates invisible spec drift.

1. Describe what doesn't line up
2. State what you think the resolution is
3. Wait for human decision before proceeding

### Stuck >15 minutes on Green

1. **Simplify** -- Implement one test at a time
2. **Add logging** -- Trace data flow
3. **Check TC** -- Is the test expectation wrong?
4. **Escalate** -- Ask human, don't spin

### Ground-Level Discovery

When implementation reveals spec gaps:

1. Document the discovery
2. Propose minimal solution
3. Get human approval
4. Update spec if significant

Don't silently deviate from spec.

### Test Failures After Green

Common causes:
- Mock setup doesn't match implementation expectations
- Type mismatch between mock and real data
- Race condition in async test

Debug by checking mock return values against what implementation actually needs.

---

## Constraints

- **Only implement what's in scope** -- Follow the story and referenced artifacts; no "improvements" or extra features
- **Match existing patterns** -- Look at how similar things are done in the codebase
- **No hardcoding for tests** -- Implement actual logic that works for all valid inputs
- **Readable code** -- Clear names, maintainable solutions, comments only where logic isn't self-evident

---

## Common Issues

### Tests pass but behavior is wrong
**Cause:** Tests don't assert actual behavior
**Fix:** Review tests -- are they checking real outcomes?

### Tests ERROR instead of PASS after Green
**Cause:** Stubs still throwing somewhere
**Fix:** Check all code paths hit by tests are implemented

### Tests FAIL (not ERROR) after Green
**Cause:** Implementation doesn't match expected behavior
**Fix:** Debug the specific assertion failure

### Previous tests break
**Cause:** Regression introduced
**Fix:** Don't proceed -- fix the regression first

### Type errors in tests
**Cause:** Mock types don't match real types
**Fix:** Ensure fixtures match actual data shapes

---

## Output Format

After completing each phase, summarize:
- Files created/modified
- Test counts (new, total)
- Any decisions made
- Any issues discovered
- Confirmation of expected state (X tests PASS, Y tests ERROR)

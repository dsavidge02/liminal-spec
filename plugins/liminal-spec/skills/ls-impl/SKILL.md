---
name: ls-impl
description: Implement stories using TDD discipline. Plan-mode compatible. Reads complete stories with functional and technical sections.
---

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

**Reference artifacts** (in the project repo, read as needed):
- **Epic** -- full functional requirements, data contracts, user flows
- **Tech Design** -- architecture, all interfaces, module boundaries, verification scripts

The story is your primary guide. The reference artifacts provide depth when you need it -- the full interface definition when the story excerpts aren't enough, the sequence diagram when you need to understand a flow end-to-end.

### Plan Before You Build

Before writing code, form an implementation plan:

1. Read the story's functional and technical sections
2. Review relevant epic flows and tech design sections for context
3. Identify the implementation sequence (which modules in what order)
4. Note any risks or ambiguities from the story's Risks & Constraints
5. Check the spec deviation field -- know where this story diverges from the design

If using plan mode, present your plan for approval before executing. The story is the planning input -- it should give you everything you need to form a concrete plan.

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

---

## Reference: confidence-chain

## The Confidence Chain

Every line of code traces back through a chain:

```
AC (requirement) → TC (test condition) → Test (code) → Implementation
```

**Validation rule:** Can't write a TC? The AC is too vague. Can't write a test? The TC is too vague.

This chain is what makes the methodology traceable. When something breaks, you can trace from the failing test back to the TC, back to the AC, back to the requirement.

---

## Reference: Verification: The Scrutiny Gradient

# Verification: The Scrutiny Gradient

**Upstream = more scrutiny. Errors compound downward.**

The epic gets the most attention because if it's on track, everything else follows. If it's off, everything downstream is off.

## The Gradient

```
Epic:  #################### Every line
Tech Design:   #############....... Detailed review
Stories:       ########............ Key things + shape
Implementation:####................ Spot checks + tests
```

## Epic Verification (MOST SCRUTINY)

This is the linchpin. Read and verify EVERY LINE.

### Verification Steps

1. **BA self-review** -- Critical review of own work. Fresh eyes on what was just written.

2. **Tech Lead validation** -- Fresh context. The Tech Lead validates the spec is properly laid out for tech design work:
   - Can I map every AC to implementation?
   - Are data contracts complete and realistic?
   - Are there technical constraints the BA missed?
   - Do flows make sense from implementation perspective?

3. **Additional model validation** -- Another perspective (different model, different strengths):
   - Different model, different strengths
   - Adversarial/diverse perspectives catch different issues

4. **Fix all issues, not just blockers** -- Severity tiers (Critical/Major/Minor) set fix priority order, not skip criteria. Address all issues before handoff. Minors at the spec level compound downstream -- zero debt before code exists.

5. **Validation rounds** -- Run validation until no substantive changes are introduced, typically 1-3 rounds. The Tech Lead also validates before designing -- a built-in final gate. Number of rounds is at the user's discretion.

6. **Human review (CRITICAL)** -- Read and parse EVERY LINE:
   - Can you explain why each AC matters?
   - No "AI wrote this and I didn't read it" items
   - This is the document that matters most

## Tech Design Verification

Still detailed review, but less line-by-line than epic.

### What to Check

- Structure matches methodology expectations
- TC-to-test mapping is complete
- Interface definitions are clear
- Phase breakdown makes sense
- No circular dependencies

### Who Validates

- **Tech Lead self-review** -- Critical review of own work
- **BA/SM validation** -- Can I shard stories from this? Can I identify coherent AC groupings?
- **Tech Lead re-validation** -- Can I add story-level technical sections from this?

## Story Verification

Stories go through a two-phase validation reflecting their two-phase authoring.

### Functional Stories (after BA/SM sharding)

Less line-by-line, more shape and completeness:

- Coverage gate: every AC/TC assigned to a story
- Integration path trace: no cross-story seam gaps
- Each story coherent and independently acceptable
- Tech Lead confirms they can add technical sections

### Technically Enriched Stories (after Tech Lead enrichment)

Story contract compliance check:

1. **TC-to-test mapping present** -- every TC mapped to a test approach
2. **Technical DoD present** -- specific verification commands
3. **Spec deviation field present** -- even when empty
4. **Targets, not steps** -- technical sections describe what, not how

Consumer gate: could an engineer implement from this story without clarifying questions?

## Implementation Verification

Spot checks + automated tests.

### What to Check

- Tests pass (full suite)
- Types check clean
- Lint passes
- Spot check implementation against tech design
- Gorilla testing catches "feels wrong" moments

---

## Multi-Agent Validation Pattern

Liminal Spec uses this pattern throughout:

| Artifact | Author Reviews | Consumer Reviews |
|----------|---------------|------------------|
| Epic | BA self-review | Tech Lead (needs it for design) |
| Tech Design | Tech Lead self-review | BA/SM (needs it for story sharding) + Tech Lead (needs it for technical sections) |
| Functional Stories | BA/SM self-review | Tech Lead (needs them for technical enrichment) |
| Complete Stories | Tech Lead self-review | Engineer (needs them for implementation) |

### Why This Works

1. **Author review** -- Catches obvious issues, forces author to re-read
2. **Consumer review** -- Downstream consumer knows what they need from the artifact
3. **Different model** -- Different strengths catch different issues. Use adversarial/diverse perspectives for complementary coverage.
4. **Fresh context** -- No negotiation baggage, reads artifact cold

### The Key Pattern: Author + Downstream Consumer

If the Tech Lead can't build a design from the epic -> spec isn't ready.
If the BA/SM can't shard stories from tech design -> design isn't ready.
If the Tech Lead can't add technical sections to stories -> stories aren't ready.
If the Engineer can't implement from complete stories -> stories aren't ready.

**The downstream consumer is the ultimate validator.**

---

## Orchestration

**How to run validation passes is left to the practitioner.** This skill describes:
- **WHAT to validate** -- Which artifacts, which aspects
- **WHEN to validate** -- Checkpoints in the flow

Leaves flexible:
- **HOW to validate** -- Which models, how many passes
- **Specific orchestration** -- Based on your setup and preferences

---

## Checkpoints

### Before Tech Design

- [ ] Epic complete
- [ ] BA self-review done
- [ ] Model validation complete
- [ ] All issues addressed (Critical, Major, and Minor)
- [ ] Validation rounds complete
- [ ] Tech Lead validated: can design from this
- [ ] Human reviewed every line

### Before Story Sharding

- [ ] Tech Design complete (all altitudes: system context, modules, interfaces)
- [ ] Tech Lead self-review done (completeness, richness, writing quality, readiness)
- [ ] Model validation complete (different model for diverse perspective)
- [ ] All issues addressed (Critical, Major, and Minor)
- [ ] Validation rounds complete (no substantive changes remaining)
- [ ] TC -> Test mapping complete (every TC from epic maps to a test)
- [ ] BA/SM validated: can shard stories from this
- [ ] Human reviewed structure and coverage

### Before Implementation

- [ ] Functional stories complete (all ACs/TCs assigned, integration path traced)
- [ ] Technical enrichment complete (all four story contract requirements met)
- [ ] Consumer gate passed: engineer can implement from stories
- [ ] Different model reviewed stories (if high-stakes)

### Before Ship

- [ ] All tests pass
- [ ] Gorilla testing complete
- [ ] Verification checklist passes
- [ ] Human has seen it work

---

## Reference: Testing Reference

# Testing Reference

## Philosophy: Service Mocks

**Service mocks** are in-process tests at public entry points. They test as close to where external calls enter your code as possible, exercise all internal pathways, and mock only at external boundaries. Not unit tests (too fine-grained, mock internal modules). Not end-to-end tests (too slow, require deployed systems). Service mocks hit the sweet spot.

### Core Principle

Test at the entry point. Exercise the full component. Mock only what you must.

```
Your Code
┌─────────────────────────────────────────────────────┐
│  Entry Point (API handler, exported function, etc.) │ ← Test here
│         ↓                                           │
│  Internal logic, state, transformations             │ ← Exercised, not mocked
│         ↓                                           │
│  External boundary (network, DB, filesystem)        │ ← Mock here
└─────────────────────────────────────────────────────┘
```

### Why Service Mocks Work

Traditional unit tests mock at module/class boundaries — testing `UserService` by mocking `UserRepository`. This hides integration bugs between your own components.

Service mocks push the mock boundary outward to where your code ends and external systems begin. You test real integration between your modules while keeping tests fast and deterministic.

**The insight:** Your code is one unit. External systems are the boundary.

### Mock Strategy

| Boundary | Mock? | Why |
|----------|-------|-----|
| **Off-machine** (network, external APIs, services) | Always | Speed, reliability, no external dependencies |
| **On-machine, out-of-process** (local database, Redis) | Usually | Speed; judgment call based on setup complexity |
| **In-process** (your code, your modules) | Never | That's what you're testing |

### The Two Test Layers

Coverage comes from two complementary layers:

**Layer 1: Service mocks (primary)**
- Many tests, fast, in-process
- This is where TDD lives
- Coverage goals met here
- Run on every save, every CI build

**Layer 2: Wide integration tests (secondary)**
- Few tests, slower, require deployed environment
- Verify deployed pieces work together
- Catch configuration and wiring issues
- Run locally before merge, post-CD as verification — NOT on CI

```
┌──────────────────────────────────────────────────┐
│  Service Mocks (many, fast, in-process)         │  ← TDD lives here
│  Coverage goals met here                         │
└──────────────────────────────────────────────────┘
                        +
┌──────────────────────────────────────────────────┐
│  Wide Integration Tests (few, slower, deployed)  │  ← Smoke tests, critical paths
│  Run locally + post-CD, not CI                   │
└──────────────────────────────────────────────────┘
```

### Confidence Distribution

Service mocks provide high confidence for logic and behavior. Wide integration tests provide confidence for deployment and wiring. Together they cover most failure modes.

**What they can't cover:** Visual correctness, UX feel, edge cases you didn't anticipate. That's what gorilla testing is for.

---

## API Testing (Deepest Section)

API testing is the cleanest application of service mocks. The entry point is obvious (the HTTP handler), the boundaries are clear (external services), and the response is easily asserted. This is the pattern to internalize — UI testing adapts it with more friction.

### Pattern: Test the Route Handler

Get as close to the HTTP handler as possible. Use your framework's test injection (Fastify's `inject()`, Express's supertest, etc.) to send requests without network overhead.

```typescript
// Service mock test for POST /api/prompts
describe("POST /api/prompts", () => {
  let app: FastifyInstance;

  beforeEach(async () => {
    app = buildApp();  // Your app factory
    await app.ready();
  });

  afterEach(async () => {
    await app.close();
  });

  describe("authentication", () => {
    // TC-1: requires authentication
    test("returns 401 without auth token", async () => {
      const response = await app.inject({
        method: "POST",
        url: "/api/prompts",
        payload: { prompts: [] },
      });

      expect(response.statusCode).toBe(401);
    });
  });

  describe("validation", () => {
    // TC-2: validates input
    test("returns 400 with invalid slug format", async () => {
      const response = await app.inject({
        method: "POST",
        url: "/api/prompts",
        headers: { authorization: `Bearer ${testToken()}` },
        payload: {
          prompts: [{ slug: "Invalid:Slug", name: "Test", content: "Test" }],
        },
      });

      expect(response.statusCode).toBe(400);
      expect(response.json().error).toMatch(/slug/i);
    });
  });

  describe("success paths", () => {
    // TC-3: creates prompt and returns ID
    test("persists to database and returns created ID", async () => {
      mockDb.insert.mockResolvedValue({ id: "prompt_123" });

      const response = await app.inject({
        method: "POST",
        url: "/api/prompts",
        headers: { authorization: `Bearer ${testToken({ sub: "user_1" })}` },
        payload: {
          prompts: [{ slug: "my-prompt", name: "My Prompt", content: "Content" }],
        },
      });

      expect(response.statusCode).toBe(201);
      expect(response.json().ids).toContain("prompt_123");
      expect(mockDb.insert).toHaveBeenCalledWith(
        expect.objectContaining({ slug: "my-prompt", userId: "user_1" })
      );
    });
  });

  describe("error handling", () => {
    // TC-4: handles database errors gracefully
    test("returns 500 when database fails", async () => {
      mockDb.insert.mockRejectedValue(new Error("Connection lost"));

      const response = await app.inject({
        method: "POST",
        url: "/api/prompts",
        headers: { authorization: `Bearer ${testToken()}` },
        payload: { prompts: [{ slug: "test", name: "Test", content: "Test" }] },
      });

      expect(response.statusCode).toBe(500);
      expect(response.json().error).toMatch(/internal/i);
    });
  });
});
```

### Setting Up Mocks

Mock external dependencies before importing the code under test. The pattern is framework-agnostic:

```typescript
// Mock external boundaries — database, auth service, config
const mockDb = {
  insert: vi.fn(),
  query: vi.fn(),
  delete: vi.fn(),
};
vi.mock("../lib/database", () => ({ db: mockDb }));

vi.mock("../lib/auth", () => ({
  validateToken: vi.fn(async (token) => {
    if (token === "valid") return { valid: true, userId: "user_1" };
    return { valid: false };
  }),
}));

// Reset between tests
beforeEach(() => {
  vi.clearAllMocks();
});
```

### What Makes a Good API Service Mock Test

1. **Tests one behavior** — authentication, validation, success path, or error handling
2. **Uses real request/response** — not calling internal functions directly
3. **Mocks only external boundaries** — database, auth service, external APIs
4. **Asserts on observable behavior** — status code, response body, side effects
5. **Traces to a TC** — comment links back to spec

### Wide Integration Tests for APIs

After service mocks verify logic, wide integration tests verify the deployed system works:

```typescript
// Integration test — runs against deployed staging
describe("Prompts API Integration", () => {
  const baseUrl = process.env.TEST_API_URL;
  let authToken: string;

  beforeAll(async () => {
    authToken = await getTestAuth();
  });

  test("create and retrieve prompt round trip", async () => {
    const slug = `test-${Date.now()}`;

    // Create
    const createRes = await fetch(`${baseUrl}/api/prompts`, {
      method: "POST",
      headers: { Authorization: `Bearer ${authToken}`, "Content-Type": "application/json" },
      body: JSON.stringify({ prompts: [{ slug, name: "Test", content: "Test" }] }),
    });
    expect(createRes.status).toBe(201);

    // Retrieve
    const getRes = await fetch(`${baseUrl}/api/prompts/${slug}`, {
      headers: { Authorization: `Bearer ${authToken}` },
    });
    expect(getRes.status).toBe(200);
    expect((await getRes.json()).slug).toBe(slug);

    // Cleanup
    await fetch(`${baseUrl}/api/prompts/${slug}`, {
      method: "DELETE",
      headers: { Authorization: `Bearer ${authToken}` },
    });
  });
});
```

**When to run:**
- Locally before merge
- Post-CD as deployment verification
- NOT on CI (too slow, requires deployed environment)

---

## UI Testing (Lighter Section)

UI testing follows the same service mock philosophy but with more friction. The "entry point" is less clear, browser APIs complicate mocking, and visual/UX correctness can't be verified programmatically.

**Same ideals, messier execution.** UI tests can't match API test confidence. Aim for behavioral coverage, then rely on gorilla testing for visual/UX verification.

### The Principle Applied to UI

Mock at the API layer (fetch calls, API client). Let UI framework internals (state, hooks, DOM updates) run for real. Test user interactions and their effects.

```
UI Code
┌─────────────────────────────────────────────────────┐
│  User Interaction (click, type, submit)             │ ← Simulate here
│         ↓                                           │
│  Component logic, state, framework internals        │ ← Runs for real
│         ↓                                           │
│  API calls (fetch, client library)                  │ ← Mock here
└─────────────────────────────────────────────────────┘
```

### HTML/JS (No Framework)

For plain HTML with JavaScript, use jsdom to load templates and test behavior:

```typescript
import { JSDOM } from "jsdom";

describe("Prompt Editor", () => {
  let dom: JSDOM;
  let fetchMock: vi.Mock;

  beforeEach(async () => {
    dom = await JSDOM.fromFile("src/prompt-editor.html", { runScripts: "dangerously" });
    fetchMock = vi.fn(() => Promise.resolve({ ok: true, json: () => ({ id: "new_id" }) }));
    dom.window.fetch = fetchMock;
  });

  // TC-3: Submit valid form creates prompt
  test("submitting form calls POST /api/prompts", async () => {
    const doc = dom.window.document;
    doc.getElementById("slug").value = "new-prompt";
    doc.getElementById("name").value = "New Prompt";
    doc.getElementById("prompt-form").dispatchEvent(new dom.window.Event("submit"));

    await new Promise((r) => setTimeout(r, 50));

    expect(fetchMock).toHaveBeenCalledWith("/api/prompts", expect.objectContaining({ method: "POST" }));
  });
});
```

### React / Component Frameworks

Same principle — mock API layer, let framework run for real:

```typescript
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

// Mock API layer, NOT hooks or components
vi.mock("@/api/promptApi");

describe("PromptList", () => {
  // TC-7: displays prompts when loaded
  test("renders prompt list from API", async () => {
    mockPromptApi.getAll.mockResolvedValue([{ id: "1", name: "Prompt 1" }]);

    render(<PromptList />);

    await waitFor(() => {
      expect(screen.getByText("Prompt 1")).toBeInTheDocument();
    });
  });

  // TC-8: shows error on failure
  test("displays error when API fails", async () => {
    mockPromptApi.getAll.mockRejectedValue(new Error("Failed"));

    render(<PromptList />);

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

### E2E for Critical Paths (Playwright)

E2E tests serve as wide integration for UI — verify the full deployed stack works:

```typescript
test("user can create and view prompt", async ({ page }) => {
  await page.goto("/prompts");
  await page.click('[data-testid="new-prompt-button"]');
  await page.fill('[data-testid="slug-input"]', "e2e-test");
  await page.fill('[data-testid="name-input"]', "E2E Test");
  await page.click('[data-testid="submit-button"]');

  await expect(page).toHaveURL(/\/prompts\/e2e-test/);
  await expect(page.getByText("E2E Test")).toBeVisible();
});
```

Run locally and post-CD, not on CI.

### UI Testing Limitations

**Acknowledge the gap:** UI testing cannot match API testing confidence. Visual correctness, UX polish, interaction feel — not verifiable programmatically.

Plan for more gorilla testing. Plan for iterative polish. The GORILLA phase exists partly for this.

---

## CLI Testing

For CLI tools, the entry point is the command handler. The same service mock principle applies: test at the entry point, exercise internal modules through it, mock only at external boundaries (filesystem, network, child processes).

### The Principle Applied to CLI

```
CLI Code
┌─────────────────────────────────────────────────────┐
│  Command handler (yargs, commander, etc.)           │ ← Test here
│         ↓                                           │
│  Internal orchestration (executors, managers)        │ ← Exercised, not mocked
│         ↓                                           │
│  Pure algorithms (parsing, transforming)             │ ← Can test directly (no mocks needed)
│         ↓                                           │
│  Filesystem / network / child processes             │ ← Mock here
└─────────────────────────────────────────────────────┘
```

| Layer | Mock? | Why |
|-------|-------|-----|
| Command handler | Test here | Entry point |
| Internal orchestration (executors, managers) | Don't mock | Exercise through command |
| Pure algorithms (no IO) | Can test directly | No mocking needed, supplemental coverage |
| Filesystem / network / child processes | Mock | External boundary |

### Correct Structure

```
tests/
├── commands/              # Entry point tests (primary coverage)
│   ├── edit-command.test.ts    # Full edit flow, mocks filesystem
│   ├── clone-command.test.ts   # Full clone flow, mocks filesystem
│   └── list-command.test.ts    # Full list flow, mocks filesystem
└── algorithms/            # Pure function tests (supplemental)
    └── tool-call-remover.test.ts  # No mocks, edge case coverage
```

### Anti-Pattern

```
tests/
├── edit-operation-executor.test.ts  # ❌ Internal module with mocked fs
├── backup-manager.test.ts           # ❌ Internal module with mocked fs
├── tool-call-remover.test.ts        # ✓ Pure algorithm, ok
└── edit-command.test.ts             # ✓ Entry point, ok
```

The anti-pattern tests internal modules in isolation with mocked dependencies. This hides integration bugs between your own components — exactly what service mocks avoid. An agent seeing API and UI examples ("test the route handler," "test the component") will pattern-match to "test the executor, test the manager" unless given explicit CLI guidance.

---

## Convex Testing

Convex functions are serverless handlers. Same service mock principle — mock external boundaries, test the function directly:

```typescript
describe("withApiKeyAuth wrapper", () => {
  beforeEach(() => {
    process.env.CONVEX_API_KEY = "test_key";
  });

  test("validates API key and calls handler", async () => {
    const handler = vi.fn(async (ctx, args) => ({ userId: args.userId }));
    const wrapped = withApiKeyAuth(handler);

    const result = await wrapped({}, { apiKey: "test_key", userId: "user_1" });

    expect(result.userId).toBe("user_1");
    expect(handler).toHaveBeenCalled();
  });

  test("rejects invalid API key", async () => {
    const wrapped = withApiKeyAuth(vi.fn());

    await expect(wrapped({}, { apiKey: "wrong", userId: "user_1" })).rejects.toThrow("Invalid");
  });
});
```

---

## TC Traceability

Every test must trace to a Test Condition from the Epic. This is the Confidence Chain in action.

### In Test Code

```typescript
describe("POST /api/prompts", () => {
  // TC-1: requires authentication
  test("returns 401 without auth token", async () => { ... });

  // TC-2: validates slug format
  test("returns 400 with invalid slug", async () => { ... });
});
```

### In Test Plan

| TC ID | Test File | Test Name | Status |
|-------|-----------|-----------|--------|
| TC-1 | createPrompts.test.ts | returns 401 without auth token | Passing |
| TC-2 | createPrompts.test.ts | returns 400 with invalid slug | Passing |

**Rules:**
- TC ID in comment or test name
- Every TC from spec must have at least one test
- Can't write a test? TC is too vague — return to spec

---

## Anti-Patterns

### Asserting on NotImplementedError

```typescript
// ❌ Passes before AND after implementation
it("throws not implemented", () => {
  expect(() => createPrompt(data)).toThrow(NotImplementedError);
});

// ✅ Tests actual behavior
it("creates prompt and returns ID", async () => {
  const result = await createPrompt(data);
  expect(result.id).toBeDefined();
});
```

### Over-Mocking

```typescript
// ❌ Mocking your own code hides bugs
vi.mock("../hooks/useFeature");
vi.mock("../components/FeatureList");

// ✅ Mock only external boundaries
vi.mock("../api/featureApi");
```

### Testing Implementation Details

```typescript
// ❌ Internal state
expect(component.state.isLoading).toBe(true);

// ✅ Observable behavior
expect(screen.getByTestId("loading")).toBeInTheDocument();
```

---

## Test Organization

```
tests/
├── service/           # Service mock tests (primary)
│   ├── api/
│   │   └── prompts.test.ts
│   └── ui/
│       └── prompt-editor.test.ts
├── integration/       # Wide integration tests
│   ├── api.test.ts
│   └── ui.test.ts
└── fixtures/
    └── prompts.ts
```

Track running totals across stories. Previous tests must keep passing — regression = stop and fix.

---

## Reference: model-selection

## Model Selection

Different models have different strengths. Choosing the right model for a task improves both quality and efficiency.

**General principles:**

- **Capable models** work well for implementation, orchestration, and writing. They handle context, synthesize across documents, and make pragmatic decisions.
- **Detail-oriented models** work well for verification, code review, and spec auditing. They catch what builders miss -- signature mismatches, spec drift, edge cases.
- **Using multiple models** for validation provides complementary coverage. Author + different model catches more issues than author alone.

Leave specific model choice to the practitioner. The principle matters more than the specific tool: match model strengths to task requirements.

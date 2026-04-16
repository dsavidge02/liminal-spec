# Proposal: `ls-ui-design` Skill

**Status:** Proposal — pending review  
**Date:** 2026-04-16  
**Authors:** Daniel Savidge (owner), Claude Opus 4.6 (analysis + drafting), GitHub Copilot (independent analysis)

---

## Problem Statement

The Liminal Spec pipeline produces high-quality functional specifications (epics with ACs/TCs) and technical specifications (tech designs with architecture, interfaces, and test mapping), but the implemented frontend consistently falls short in two ways:

1. **Visual quality.** The resulting UI lacks professional design — inconsistent spacing, poor visual hierarchy, generic component usage, and no intentional aesthetic direction.
2. **AC coverage in the frontend.** The implemented frontend misses parts of the acceptance criteria, particularly around state coverage (loading, empty, error, validation), interaction behavior, and responsive design.

These are recurring problems observed across multiple epics and implementation runs, not one-off execution failures.

### Root Cause

The pipeline has no artifact dedicated to visual and interaction specification. The confidence chain (AC -> TC -> Test -> Implementation) verifies that DOM elements *exist* but says nothing about layout, spacing, visual hierarchy, responsiveness, or interaction polish.

- The **Epic** defines user behavior — flows, ACs, TCs, boundary contracts. It does not own screen composition, visual language, or interaction micro-behavior.
- The **Tech Design** defines module architecture, interfaces, test mapping, and implementation structure. Even when split into Config B with a `tech-design-client.md` companion, it does not own screen-level UX intent, visual coherence, or state presentation rules.
- **Implementation agents** receive stories with behavioral ACs like "page displays location list when fetch succeeds." This is testable as a unit test (DOM element exists) but tells the implementer nothing about layout, spacing, visual polish, responsive behavior, or state transitions. Agents infer — and they infer wrong.

The gap is structural, not a matter of writing better ACs or more detailed tech designs.

---

## Analysis Process

This proposal is grounded in independent analysis from multiple models and multiple angles, followed by iterative scope refinement.

### Phase 1: Independent Multi-Agent Investigation

Three Claude Opus subagents investigated the proposal independently and in parallel, each approaching from a different angle:

**Agent 1 — Pipeline Placement Analysis.** Read the manifest, tech-design, publish-epic, epic, and both implementation orchestration skills. Analyzed handoff points between phases and identified where visual/UI intent gets lost. Conclusion: the gap is between Tech Design and Publish Epic; visual intent is never captured and stories carry no visual reference.

**Agent 2 — Gap Analysis.** Read epic, tech-design, prd, confidence-chain, writing-style-epic, and team-impl-cc to find where frontend/UI concerns are and aren't addressed. Conclusion: ACs are behavioral not visual, TCs don't surface design intent, tech design has zero mention of visual contracts or accessibility patterns, and implementation verifiers check DOM existence not visual correctness. The gap is structural.

**Agent 3 — Feasibility Analysis.** Evaluated what "mockup" means in a text-based AI workflow and what artifact formats would actually be useful to implementation agents. Conclusion: "mockup" is the wrong framing; what's needed is structured UI component specifications. Argued the existing Config B mechanism could be enhanced rather than adding a new skill.

### Phase 2: Cross-Model Analysis (Copilot)

GitHub Copilot independently produced a comprehensive analysis covering the same territory. Key contributions:

- Confirmed the structural gap diagnosis
- Identified that the pipeline was recently simplified (removal of `ls-story-tech`) — adding ceremony needs justification
- Argued strongly against "pure mockup" framing: a mockup artifact that doesn't participate in the confidence chain would be "pretty but non-binding"
- Proposed the responsibility boundary model (Epic owns behavior, Tech Design owns architecture, new skill owns visual/interaction specification)

### Phase 3: Cross-Model Synthesis

All four analyses (3 Claude + 1 Copilot) were synthesized into a unified assessment. Universal agreement on:

- The gap is real and structural
- "Mockup" is the wrong framing — structured UI specifications are what's needed
- Placement: after Tech Design, before Publish Epic
- Traceability into the confidence chain is non-negotiable
- Implementation verification must check against the UI spec or it becomes advisory

The main divergence: whether to add a new skill vs. enhance `ls-tech-design`. Three of four analyses favored a dedicated skill. The argument: if the existing Config B mechanism were sufficient, the problem wouldn't be recurring. Making the visual gate explicit and separate ensures it actually happens.

### Phase 4: Scope Refinement

Four additional scope considerations were raised and analyzed:

1. **Existing UI patterns.** The skill must be codebase-aware — it should inventory existing components, design tokens, color schemes, and page layouts before specifying anything new.
2. **Reusable component architecture.** The skill should push for design tokens, reusable components, and layout primitives where justified — but avoid performative abstraction.
3. **Frontend-design skill integration.** An established `frontend-design` skill was analyzed for transferable concepts. The quality bar and intentional design thinking transfer well; the "bold/unique every time" creative freedom does not — product UI should be consistent, not novel.
4. **Implementation skill integration.** The `ls-team-impl-cc`, `ls-team-impl`, and `ls-codex-impl` skills need downstream changes to consume and verify against the UI spec.

A fifth consideration was added: **reference material support.** Users frequently have existing mockups, screenshots, Figma exports, or sketches that represent their vision. The skill should treat these as first-class inputs that constrain the specification.

---

## Proposal

### Skill: `ls-ui-design`

**Description:** Produce a codebase-aware frontend UI design specification with screen inventory, component contracts, state coverage, and AC-to-UI traceability. Optional — invoked when frontend complexity, visual quality, or interaction correctness materially affect the outcome.

### Pipeline Position

```
Epic -> Tech Design -> [UI Design] -> Publish Epic -> Implementation
```

The brackets indicate this phase is optional. Backend-only epics skip it. Frontend-heavy or customer-facing work invokes it.

**Why this placement:**

- **After Tech Design:** The tech design provides complete inputs — module architecture, interfaces, data contracts, component boundaries, and state management decisions. Designing UI before these are settled produces speculative designs that ignore implementation reality.
- **Before Publish Epic:** Output enriches stories before they're published. Stories can reference UI spec sections in their Technical Design area. Visual ambiguity is caught before implementation starts, not during it.
- **Not parallel to Tech Design:** Splits focus when both need the same Epic inputs, creates coordination overhead, and risks misalignment.
- **Not after Publish Epic:** Too late. Story boundaries are already established. UI discoveries that reveal new states or screens would create churn back into the stories.

### Inputs

The skill consumes inputs in this priority order:

1. **Reference materials** (if provided) — mockups, screenshots, Figma exports, sketches, competitor references, or any visual artifact representing the user's vision. These are the strongest signal for visual intent and become the starting constraint when present.
2. **Existing product UI system evidence** — the codebase itself. Components, design tokens, layouts, styling approach, page patterns already in use.
3. **Epic** — flows, ACs, TCs, boundary contracts. The behavioral specification the UI must satisfy.
4. **Tech Design** and `tech-design-client.md` (if present) — module architecture, interfaces, component boundaries, state management decisions. The technical world the UI lives in.
5. **Architecture document** (if present) — platform targets, design system constraints, framework decisions.
6. **Human-provided context** — Storybook links, design docs, brand guidelines, or other reference surfaces when code evidence is incomplete.

If source-level UI artifacts exist, the skill inspects them. If only partial references exist, the skill explicitly states that its audit is limited and asks for missing evidence rather than silently inventing a system.

### Process Phases

**Phase 1: Reference Material Review** (conditional)

If the user provides reference images, mockups, or screenshots:
- Analyze each reference for layout patterns, color usage, component structure, interaction hints, and visual direction
- Catalog what was provided and what was extracted from each
- Establish the reference as the visual starting point that the rest of the spec must honor or explicitly deviate from (with rationale)

If no reference materials are provided, proceed directly to Phase 2.

**Phase 2: Design Inventory** (codebase-aware)

Scan the codebase for existing UI patterns:
- Existing component library or design system (shared components, UI primitives)
- Design tokens (CSS variables, Tailwind config, theme files, color definitions)
- Page layout patterns (shells, sidebars, headers, grid systems)
- Typography conventions (font families, heading hierarchy, body text patterns)
- State treatment patterns (how loading, error, empty, and success states are currently handled)
- Styling approach (Tailwind, CSS Modules, styled-components, etc.)
- Accessibility patterns already in use

If `ls-current-docs` output exists, use it as a head start. The design inventory is narrower and more design-focused than current-docs — different lens, but overlapping codebase scan.

**Phase 3: Visual System Strategy**

Commit to one of three strategies based on the inventory findings and any reference materials:

- **Inherit** — a mature, consistent design system exists. Spec screens using existing tokens and components. Propose additions only where gaps exist.
- **Extend** — a partial system exists (some tokens, some shared components, inconsistent patterns). Spec screens using what's there, propose specific additions that fill gaps, flag inconsistencies.
- **Establish** — no meaningful system exists. Define foundational tokens (color roles, spacing scale, typography hierarchy) and core reusable components as part of the output.

The strategy must include evidence (what was checked, what was found) and rationale. This gate prevents the skill from reinventing a system that already works or ignoring the absence of one.

When reference materials exist, the strategy also addresses how the reference vision aligns with (or departs from) the existing system.

**Phase 4: Screen-Level Specification**

The core output work:
- Screen inventory — what screens/views exist for this feature
- Flow-to-screen mapping — which epic flows map to which screens
- AC-to-screen/state traceability map — which ACs are satisfied by which screen in which state
- Per-screen state coverage — loading, empty, error, success, validation, disabled, responsive variants
- Component specifications — new and extended components with prop contracts, layout constraints, state presentations
- Layout templates with slot definitions and responsive behavior
- Interaction notes — hover, focus, transitions, navigation, destructive action treatment
- Accessibility expectations — keyboard access, focus order, ARIA labels, semantic structure, contrast

**Phase 5: Validation Before Handoff**

- Every AC with a frontend surface maps to a screen/component
- Every specified component traces to at least one AC
- State coverage matrix is complete (no unaddressed states)
- Design tokens are either existing (verified in codebase) or new (defined in the spec)
- Responsive behavior specified for all relevant breakpoints
- Component contracts are implementable (no vague directives like "make it look professional")
- Reference material fidelity verified — spec honors the reference or documents deviations with rationale

### Output Artifact

A single `ui-spec.md` containing these sections:

| # | Section | Purpose |
|---|---------|---------|
| 1 | Reference Material Analysis | What was provided, what was extracted, how it informs the spec (conditional — omitted if no references) |
| 2 | Visual System Strategy | Inherit / extend / establish decision with evidence and rationale |
| 3 | Existing UI Inventory | Components, tokens, layouts, patterns already in the codebase |
| 4 | Screen Inventory | What screens/views exist for this feature |
| 5 | AC-to-Screen/State Map | Traceability — which ACs are satisfied where, in which state |
| 6 | State Coverage per Screen | Loading, empty, error, success, validation, disabled, responsive |
| 7 | Component Specifications | New + extended components with prop contracts and state presentations |
| 8 | Component Reuse Matrix | Which screens consume which shared components |
| 9 | Proposed Reusable Additions | New components, layout shells, or token families this feature introduces |
| 10 | Token and Theme Impact | Reuse existing tokens, add new semantic tokens, or reveal token gaps |
| 11 | Layout and Responsive Rules | Layout templates, breakpoint table, per-tier behavior |
| 12 | Interaction and Motion Notes | Hover, focus, transitions, destructive actions, timing |
| 13 | Accessibility Expectations | Keyboard, focus order, ARIA, contrast, semantic structure |
| 14 | Visual Direction | Typography hierarchy, color roles, spacing rhythm, quality bar |
| 15 | Issues Found | New states or ACs discovered — feedback to epic or tech design |
| 16 | Open Questions and Assumptions | Explicit unknowns and design assumptions made |

### Frontend-Design Skill: What Transfers

The established `frontend-design` skill (analyzed at `~/.claude/skills/frontend-design/SKILL.md`) provides a strong aesthetic framework. The following concepts transfer to `ls-ui-design` in adapted form:

**Borrowed (adapted to specification):**

| Frontend-Design Concept | ls-ui-design Adaptation |
|---|---|
| Design Thinking (Purpose, Tone, Constraints, Differentiation) | Visual System Strategy gate — determine whether to inherit, extend, or establish; establish the visual direction that matches the product's existing identity |
| Typography system | Specify typography hierarchy — heading levels, body text, captions, labels — with semantic roles and size/weight relationships. Reference existing tokens or define new ones. |
| Color & theme | Specify color roles (primary, accent, error, success, surface) with semantic meaning as token contracts. Reference or extend existing palette. |
| Spatial composition | Layout templates with slot definitions, spacing rhythm as structured constraints, information hierarchy rules |
| Motion / interaction | Behavioral contracts with timing — "loading transition: skeleton -> content, fade 200ms" — not CSS implementation |
| Rejection of generic AI output | Quality bar: the spec should produce intentional, cohesive UI, not cookie-cutter defaults |

**Not borrowed:**

| Frontend-Design Concept | Why It Doesn't Transfer |
|---|---|
| "Pick a bold/extreme aesthetic direction" | Product UI should be consistent with existing identity, not novel per feature |
| "Never converge on common choices" | Consistency is a feature in product UI, not a bug |
| Specific implementation guidance (CSS vars, Motion library) | Implementation detail belongs in tech design or code, not UI specification |
| Greenfield creative freedom | The skill operates within existing product constraints by default |

### Operating Principles

The skill's behavior follows this decision hierarchy:

1. **Audit before inventing** — understand what exists before proposing anything new
2. **Reuse before creating** — use existing components and tokens before defining new ones
3. **Extend before replacing** — fill gaps in the current system rather than rebuilding it
4. **Trace every screen and state back to ACs and TCs** — nothing in the spec exists without a requirement link
5. **Document every new reusable primitive** — new components, tokens, and patterns are explicit proposals, not silent additions
6. **Raise the aesthetic bar without breaking product coherence** — intentional design within existing identity

### Shared Dependencies (Build Composition)

The skill will use existing shared files from `src/shared/`:

- `confidence-chain` — for AC-to-screen traceability methodology
- `dimensional-reasoning` — for design trade-off evaluation

New concepts (design inventory, visual system strategy, component reuse matrix) stay inline in the phase source file. Extraction to shared would be premature — no other skill currently needs them. If `ls-tech-design` later benefits from design inventory awareness, that's the right time to extract.

### Manifest Entry

```json
"ls-ui-design": {
  "name": "ls-ui-design",
  "description": "Produce a codebase-aware frontend UI design specification with screen inventory, component contracts, state coverage, and AC-to-UI traceability. Consumes epic, tech design, and optional reference materials to bridge the gap between behavioral requirements and visual implementation.",
  "phases": ["ui-design"],
  "shared": ["confidence-chain", "dimensional-reasoning"]
}
```

---

## Downstream Changes Required

The UI spec artifact is only useful if downstream skills know it exists and verify against it. The following changes are needed outside the skill itself.

### `ls-publish-epic`

Stories need to carry UI spec references in their Technical Design section. When a `ui-spec.md` exists, each story's Technical Design area should reference the relevant screens, components, and state specifications from the UI spec — the same way stories currently reference tech design sections.

### `ls-team-impl-cc`

**Artifact collection** (On Load -> Collect Artifacts): Add UI spec as an optional artifact:

```
- **UI spec** (optional) — visual specifications, component contracts,
  and AC-to-screen mapping. Present for frontend-heavy features.
```

**Reading journey** in all prompts: The UI spec slots after tech design companions and before the epic:

```
1. Tech design index
2. Tech design companions (if provided)
3. UI spec (if provided)              <- NEW
4. Epic
5. Test plan
6. Story
```

**Sonnet verifier (Prompt 3)**: Add visual compliance checking. When a UI spec exists, the verifier checks: does the implementation match the component contracts, state presentations, and layout constraints from the UI spec? This is the critical verification gap — without it, the spec exists but nobody audits against it.

**Opus verifier (Prompt 2)**: Lighter touch — verify that component boundaries in the implementation match the UI spec's component inventory and that design token usage is consistent.

**Epic-level reviewers (Prompts 4 and 5)**: Include UI spec in the full-codebase review. Check cross-story visual consistency and component reuse compliance.

### `ls-team-impl`

Same pattern as `ls-team-impl-cc`. The artifact collection adds UI spec as optional. The implementer and reviewer handoff templates include it in the reading journey. The external CLI reviewer gets the UI spec path for compliance checking.

### `ls-codex-impl`

Same pattern. UI spec added as optional artifact in the reading journey and verification.

---

## Responsibility Boundaries

Clear ownership prevents duplication and confusion between adjacent skills.

### Epic

**Owns:** user behavior, flows, acceptance criteria, test conditions, boundary contracts  
**Does not own:** screen composition details, visual language, interaction micro-behavior beyond functional expectations

### Tech Design

**Owns:** module architecture, interfaces, test mapping, technical constraints, implementation structure  
**Does not fully own:** screen-level UX intent, visual coherence, detailed state presentation rules

### UI Design (proposed)

**Owns:** screen inventory, state design, AC-to-UI mapping, component visual contracts, interaction notes, visual direction, reuse strategy, accessibility expectations  
**Does not own:** backend architecture, implementation sequencing, story publishing mechanics, TypeScript interfaces (those stay in tech design)

### Publish Epic

**Owns:** carrying relevant UI references into stories, preserving traceability from stories back to epic, tech design, and UI spec

### Overlap Zone: Tech Design Client Companion vs. UI Spec

When Config B produces a `tech-design-client.md`, there is an overlap zone with the UI spec. The boundary:

- **Tech design client companion** owns the TypeScript interface definitions, module architecture, hook contracts, state management approach, and API integration patterns for the frontend.
- **UI spec** owns the visual contracts — what those same components look like, how they present states, their layout rules, token usage, and interaction behavior.

They reference each other but do not duplicate. The tech design says "LocationList receives `LocationListProps` with these fields." The UI spec says "LocationList renders as a table with these column widths, this row spacing, these state presentations, consuming these design tokens."

---

## Risks

| Risk | Mitigation |
|---|---|
| Added ceremony in a recently simplified pipeline | Skill is optional — only invoked for frontend-heavy work |
| Duplication with tech-design-client.md | Clear responsibility boundary defined above; tech design owns interfaces, UI spec owns visual contracts |
| Spec becomes stale if not tied to stories | Publish Epic carries UI spec references into stories; implementation verifiers check against it |
| Skill drifts toward reinvention of existing patterns | Visual System Strategy gate (inherit/extend/establish) with required evidence |
| Performative abstraction — over-engineering reuse | Guardrail: abstract when repetition justifies it, leave one-offs local |
| Implementation agents ignore the UI spec | Verification prompts in team-impl-cc and team-impl explicitly check compliance |
| Reference materials create false precision | Spec documents deviations from reference with rationale; validation step verifies fidelity |

---

## Implementation Checklist

Per the project's PR Checklist for skill additions:

- [ ] Create phase source file: `src/phases/ui-design.md`
- [ ] Add skill entry to `manifest.json`
- [ ] Update `scripts/build.ts` standalone name mapping
- [ ] Update `scripts/__tests__/build.test.ts` (expected skills, content tests, standalone files)
- [ ] Update `CLAUDE.md` (output structure, phases list, skill table)
- [ ] Update `README.md` (skill table, pipeline tables)
- [ ] Update `src/README-pack.md` and `src/README-markdown-pack.md` (skill listings)
- [ ] Update cross-references in other phase source files as needed
- [ ] Downstream: update `ls-publish-epic` to carry UI spec references
- [ ] Downstream: update `ls-team-impl-cc` reading journeys and verification prompts
- [ ] Downstream: update `ls-team-impl` handoff templates
- [ ] Downstream: update `ls-codex-impl` reading journey
- [ ] Run `bun run verify` (build + validate + tests)

---

## Decision Log

Scope decisions made during the analysis conversation:

| Decision | Options Considered | Choice | Rationale |
|---|---|---|---|
| Skill name | `ls-ui-spec`, `ls-frontend-spec`, `ls-design-spec`, `ls-ui-design` | `ls-ui-design` | Parallels `ls-tech-design`, describes the artifact/activity not the document type, fits naming convention |
| Design inventory | Built into skill vs. prerequisite via ls-current-docs | Built into skill | The inventory needed is narrower and more design-focused than current-docs; skill should be self-contained with opportunistic use of current-docs if available |
| Design system opinion | Prescriptive vs. descriptive vs. adaptive | Adaptive | Inherit/extend/establish gate assesses codebase reality and responds proportionally; doesn't force a system onto projects that have one or ignore projects that don't |
| Shared dependencies | Extract new concepts to src/shared/ vs. keep inline | Keep inline, use existing shared files | No other skill currently needs the new concepts; extraction is premature. confidence-chain and dimensional-reasoning already exist and apply. |
| Reference materials | Ignore, optional input, or first-class input | First-class input | Users frequently have existing mockups/screenshots; these are the strongest signal for visual intent and should be the starting constraint when present |

---

## Related Documents

- `src/phases/tech-design.md` — Tech Design phase (Config B companion model)
- `src/phases/team-impl-cc.md` — Team Implementation CC (downstream consumer)
- `src/phases/team-impl.md` — Team Implementation CLI (downstream consumer)
- `src/phases/publish-epic.md` — Publish Epic (downstream carrier of UI references)

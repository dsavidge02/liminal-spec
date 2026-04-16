# Opus Review: `ls-ui-design` Skill Proposal

**Status:** Independent review — pending owner decision
**Date:** 2026-04-16
**Reviewer:** Claude Opus 4.7 (1M context)
**Subject:** `docs/research/ui-design-skill-proposal.md`

---

## Summary

The problem is real and the shape of the solution is mostly right, but the proposal has serious tensions with the project's own stated principles that should be addressed before building. The recommendation below is to build a smaller v1, pilot on one downstream skill, and be explicit about a verification ceiling the proposal currently overpromises against.

---

## What's strong

- **Problem diagnosis is accurate.** The structural gap (AC verifies that DOM elements exist, not how they look or behave) is real. Naming it as structural — not "write better ACs" — is the right framing and matches the project's confidence-chain reasoning.
- **Pipeline placement is sound.** After Tech Design, before Publish Epic, optional. The reasoning against parallel-to-Tech-Design and after-Publish-Epic is correct.
- **Reference materials as first-class input** is genuinely good thinking and a meaningful differentiator from a generic "UI spec" template.
- **Inherit / Extend / Establish gate** is a clean decision frame and matches the project's "audit before inventing" instinct.
- **Downstream coupling is enumerated honestly** rather than hand-waved. The proposal does not pretend the skill can be added in isolation.
- **Decision Log captures the iteration**, including alternatives considered and why each was rejected. This is the right shape for a proposal of this size.

---

## Concerns, ranked by severity

### 1. The 16-section output violates "earn complexity"

This is the largest single red flag. CLAUDE.md states:

> *"A section of equal-weight bullets with no framing paragraph is a sign something is wrong. Earn complexity — don't front-load it."*
>
> *"Don't add without approval. New sections, new shared concepts, new checklist items — propose them, don't add them. The owner curates what's in the methodology."*

The proposal front-loads 16 sections as if pre-approved. Compare with `ls-tech-design`, which is the closest analog and is materially leaner. A v1 with 6–8 sections that proves out before adding more is more aligned with how this project evolves.

### 2. The verification promise may not survive contact with reality

Code-reviewing agents can check that a `<table>` with specified columns exists, but they cannot *see* whether spacing feels right, hover states are polished, or visual hierarchy reads correctly. Without a screenshot loop or human gate, "visual compliance verification" risks giving false confidence — the spec gets checked structurally and the same quality gap persists.

This deserves to be called out as a known limitation in the proposal itself, not buried as a mitigated risk. The honest framing is: agents verify *structural* UI compliance (component exists, props match, states are present in code); visual quality (spacing, polish, rhythm, hierarchy) remains a human gate.

**Refined after GPT cross-review.** The ceiling is not permanent. If the workflow adds concrete evidence surfaces — fixed-breakpoint screenshots, named state captures, Storybook renders, snapshot tests, or accessibility verification artifacts — then visual verification can become more objective than prose review allows. The methodology should not mandate a specific tool, but the skill should require *that* a verification surface exists and that the project chooses its form. Without that, the artifact stays advisory and the gap persists.

### 3. "Optional" with permanent coupling in four downstream skills

Each of `publish-epic`, `team-impl-cc`, `team-impl`, and `codex-impl` gains conditional branches ("if UI spec exists..."). That is structural coupling whether the artifact runs or not. Every future change to those four skills now has to consider the optional artifact path.

Worth piloting in *one* downstream skill (probably `team-impl-cc`) before wiring all four. Validate that the artifact is consumable and that the verification produces signal worth the cost.

### 4. The cheaper alternative is not fully buried

Three of four analyses favored a new skill. The rationale was: "if Config B were sufficient, the problem wouldn't be recurring." But that could equally mean Config B isn't being *enforced* (no verifier prompt checks against it) rather than being structurally insufficient.

A targeted intervention — a UI checklist baked into tech-design Config B + one new verifier prompt in `team-impl-cc` — might capture 70% of the benefit at 20% of the cost. The proposal asserts the new-skill choice but does not quantitatively defend it against the smaller intervention. Worth a focused spike before committing.

### 5. Overlap zones are acknowledged but not resolved

- **vs. `ls-current-docs`:** "different lens, overlapping scan" leaves room for duplication and stale drift between two codebase audits. If a project has run `ls-current-docs` recently, what specifically does the UI inventory phase add that the current-docs output didn't capture?
- **vs. `tech-design-client.md`:** "interfaces vs. visual contracts" is a fine line in practice. `LocationList renders as a table with these column widths` arguably *is* an interface/slot concern. Real boundaries get tested at first use, and the boundary will probably need to be re-litigated then.

**Tightening after GPT cross-review.** A more enforceable contract would be one-way:

- **Tech Design owns:** component identity, module placement, TypeScript interfaces, data contracts, event/model boundaries, state management approach.
- **UI Spec owns:** visual structure, state presentation rules, layout behavior, responsive behavior, token usage, accessibility expectations, motion/interaction details.
- **UI Spec references Tech Design identifiers** rather than redefining them.

This eliminates the dual-source-of-truth risk and gives reviewers a clean rule: if the UI spec is redefining an interface, it has crossed the line.

### 6. The "Establish" path is too ambitious for a feature-level skill

Defining foundational tokens, color roles, spacing scale, typography hierarchy, and core reusable components in one pass amounts to building a design system *inside* a feature spec. That is a larger, separate activity that benefits from its own framing and stakeholders. v1 should probably handle Inherit and Extend only and explicitly punt Establish to a later, dedicated effort.

### 7. Where does it sit in the confidence chain?

AC → TC → Test → Implementation. The UI spec *decorates* the chain; it does not extend it. The proposal claims chain integration but the relationship is parallel — ACs remain authoritative, and the UI spec adds presentation context to AC-affected surfaces.

This is a precision issue, not a fatal flaw. But it is worth being precise: the AC stays upstream, the UI spec annotates the AC's frontend surface, and traceability flows AC → screen/state, not AC → UI spec → screen.

### 8. No concrete failure incidents cited

"Recurring problems observed across multiple epics" — which ones, what specifically went wrong? Methodology benefits from being grounded in named incidents. The project's own writing-style-epic guidance reflects this discipline. Without specific examples, scope decisions ("we need 16 sections because…") are harder to evaluate, and it's harder to know later whether the skill actually fixed the problem it was meant to fix.

### 9. Some borrowed concepts feel like padding

The frontend-design skill comparison is interesting in framing, but the parts that transfer (typography, color, spacing) are largely codified design-system thinking, not specific to that skill. The transfer table is useful as analysis but probably doesn't need to land in the final skill content.

### 10. `ls-team-spec` orchestration is missing from the proposal's checklist

*Added after GPT cross-review.* This is the single largest omission, and one I missed in my first pass. `ls-team-spec` orchestrates the full spec pipeline and currently assumes the sequence `Epic → Tech Design → Publish Epic` in its phase transitions, prompt maps, reading journeys, and human review checkpoints. Adding a UI Design phase changes that shape. The proposal's downstream-changes section names four implementation skills but does not name `ls-team-spec`. Without that update, even a successful pilot in `team-impl-cc` would leave the orchestrated spec workflow inconsistent with the published methodology.

The README's pipeline tables, `src/README-pack.md`, and `src/README-markdown-pack.md` would also need coordinated updates.

### 11. The "optional" gate needs an invocation rubric, not just a principle

*Added after GPT cross-review.* "Frontend-heavy or customer-facing work" is a principle, not an entry rule. Without crisp triggers, the skill will be invoked inconsistently — skipped when it's needed, invoked when it isn't, or treated as a polish pass instead of a requirements artifact. A short rubric (two or three triggers, not six) would make the gate operable. Candidate triggers:

- the feature introduces net-new screens or major screen re-composition
- the feature adds or materially changes reusable UI primitives or token families
- the user provides reference materials (mockups, screenshots, Figma exports) that must be honored

A longer rubric becomes its own ceremony. Three triggers is enough to make the gate crisp.

### 12. The referenced supporting analysis file does not exist

*Added after GPT cross-review.* The proposal cites `docs/research/frontend-ui-design-skill-analysis.md` in its Related Documents section, but that file is not present in the workspace. Either add the file or remove the reference. As written, the review trail has a dangling pointer.

---

## Recommendation

Build it, but smaller and in two phases.

### v1: minimal viable UI spec

Six to eight sections, not sixteen:

1. Reference Material Analysis (conditional)
2. Visual System Strategy (Inherit / Extend only — defer Establish)
3. Existing UI Inventory
4. Screen Inventory
5. AC-to-Screen/State Map
6. State Coverage per Screen
7. Component Specifications (fold prop contracts, reuse notes, and token usage into per-component entries rather than separate sections)
8. Open Questions and Assumptions

Drop, for v1: Component Reuse Matrix, Proposed Reusable Additions, Token and Theme Impact, Layout and Responsive Rules, Interaction and Motion Notes, Visual Direction, Issues Found — fold their essentials into Component Specs or surface them on demand. Re-add as separate sections only when the lack of one is causing real downstream errors.

### Wire only `team-impl-cc` first

Pilot on one real frontend epic. Measure whether implementation quality actually improves and whether visual verification gives meaningful signal. Then decide on `team-impl`, `codex-impl`, and `publish-epic` based on evidence, not symmetry.

### Be explicit about the verification ceiling

Document in the skill itself: agents check structural compliance (component exists, props match, states are present in code). Visual quality (spacing, polish, hierarchy) remains a human gate. This is honest and prevents the spec from being treated as a substitute for design review.

### Narrow the "Establish" path (revised after GPT cross-review)

Originally I recommended deferring Establish entirely. GPT's qualification is fair: projects with immature UI systems still need *something*, and full deferral leaves them unsupported. Narrow rather than defer:

- v1 Establish path documents the *minimum required additions* and *system gaps* observed during inventory.
- v1 Establish does **not** attempt to define a full design system (token taxonomy, full type scale, complete component library).
- A separate, dedicated skill (potentially `ls-design-system`) handles full design-system establishment when the need is real.

This preserves the ability to handle immature systems without turning a feature-level skill into a design-system program.

### Require a verification surface (added after GPT cross-review)

The skill must require *that* a verification surface exists when a UI spec is produced. The form is project-chosen — fixed-breakpoint screenshots, Storybook state renders, snapshot tests, accessibility verification artifacts, or another evidence form the project commits to. This is what converts the spec from advisory to enforceable. Without it, the verification ceiling stays where Concern #2 describes it.

### Add an invocation rubric (added after GPT cross-review)

Trim Copilot's six-clause trigger list to a sharp three (see Concern #11). Phase entry needs to be crisp, but the rubric should not become its own ceremony.

### Add `ls-team-spec` and pipeline docs to the downstream checklist (added after GPT cross-review)

The proposal's checklist names four implementation skills. It must also include `ls-team-spec`, the README pipeline tables, and the pack READMEs. See Concern #10.

### Spike the tech-design Config B alternative first (~30 min)

Before committing to a new skill, evaluate: would adding a UI specification checklist to tech-design Config B + one new verifier prompt close most of the gap? If yes, the new skill may not be needed. If no, the spike will have produced concrete evidence that strengthens the case for the new skill.

### Cite at least two named incidents

Pick two recent epics where the structural gap caused observable failure. Reference them in the problem statement. They become the test cases for whether the skill actually solved what it was built to solve.

---

## Open question for the owner

The proposal as written is internally consistent and well-argued. The core question is not *whether* to address the gap — the gap is real — but *how heavy* the intervention should be, and whether to start as a new skill or as a Tech Design companion. The choice is roughly:

- **Heavy:** new skill with 16-section artifact, four downstream integrations, full Establish path. High up-front cost, full structural separation, risk of ceremony.
- **Light:** tech-design Config B checklist + one verifier prompt. Low cost, lower ceiling, risks repeating the original "spec exists but nobody enforces it" failure mode.
- **Middle-A (Opus original):** new skill with 6–8 sections, one downstream integration, Inherit/Extend only, honest verification ceiling. Pilot, measure, expand.
- **Middle-B (GPT preference, revised after cross-review):** add `ui-spec.md` first as a sanctioned, conditional Tech Design *companion* — same artifact shape, same verification surface requirement, same invocation rubric, same one-way ownership contract. Promote to a distinct skill and pipeline phase only if the companion path proves too easy to skip or too easy to blur into architecture.

After cross-review, **Middle-B is probably the better default.** It captures the same artifact and the same enforceability while avoiding the largest costs of a new phase: the `ls-team-spec` orchestration update, the pipeline doc updates, the standalone pack reordering, and the conditional branches in four implementation skills. If teams skip the companion or blur it into architecture, that is concrete evidence that the explicit gate is needed — and at that point the promotion to a top-level skill is justified rather than asserted.

The heavy path may still be the right destination eventually. Jumping to it now skips the evidence that would justify its weight.

---

## Related Documents

- `docs/research/ui-design-skill-proposal.md` — the proposal under review
- `docs/research/frontend-ui-design-skill-analysis.md` — Copilot's independent analysis
- `CLAUDE.md` — project principles cited above (Earn complexity, Don't add without approval, Audit before inventing)

---

## GPT Cross-Verification Commentary

The following commentary reflects GitHub Copilot's independent comparison of this Opus review against its own review in `docs/research/ui-design-skill-gpt-review.md`.

### Where GPT agrees with Opus

- **The core diagnosis is correct.** The current workflow is materially stronger at behavioral and technical specification than it is at screen-level UI intent, state presentation, and visual consistency.
- **The proposed placement is right** if this becomes a distinct artifact: after Tech Design and before Publish Epic.
- **The verification story is overstated in the proposal as written.** Without a stronger evidence loop, agents can verify structural UI compliance better than they can verify polish, hierarchy, or visual quality.
- **The cheaper alternative is still live.** The proposal has not fully disproven the lighter-weight option of producing a UI-spec artifact as part of Tech Design rather than introducing a new top-level phase immediately.
- **The overlap with Tech Design is real.** Without a stricter ownership boundary, this proposal risks creating a dual source of truth around components.
- **The proposal is too heavy in its current form.** It should be tightened before implementation.
- **Concrete failure incidents would strengthen the case.** The proposal would be more persuasive if it named specific recent epics where the structural gap caused observable failure.
- **The frontend-design comparison is more useful as proposal analysis than as final methodology content.**

### Where GPT disagrees with Opus, or would qualify the point differently

#### 1. The 16-section output is not the main blocker

GPT agrees that the proposal is too heavy, but does not see the 16-section shape as the deepest problem by itself. The larger issues are:

- incomplete workflow integration
- insufficiently objective verification
- unresolved ownership boundaries with Tech Design
- an under-argued decision to create a new top-level skill instead of starting with a Tech Design companion artifact

If those issues were solved, section count would still matter, but it would be a secondary design problem rather than the primary blocker.

#### 2. The verification ceiling should not be treated as permanently fixed

Opus argues that agents can verify structural compliance while visual quality remains a human gate. GPT agrees that this is true for the proposal as written.

However, GPT would qualify the claim more narrowly. If the workflow adds stronger evidence surfaces such as fixed-breakpoint screenshots, named state captures, or similar runtime artifacts, then some aspects of visual verification can become more objective than the Opus review suggests. So the limitation is real, but may be more a limitation of the current proposal than a hard ceiling of the methodology forever.

#### 3. GPT is less ready to endorse “new skill, smaller v1” as the default path

Opus recommends a middle path: smaller new skill, one downstream integration, and a pilot.

GPT sees that as reasonable, but still prefers an even lower-risk default: pilot the `ui-spec.md` artifact first as a conditional Tech Design companion. Then promote it to a distinct phase only if that lighter path proves too easy to skip or too easy to blur into architecture work.

This is the main strategic difference in emphasis between the two reviews.

#### 4. GPT would narrow the Establish path rather than fully defer it

Opus recommends deferring the Establish path entirely.

GPT agrees that the current Establish path is too ambitious, but would more likely narrow it to something like: document minimum required additions and system gaps, rather than attempt to define a full design system. That preserves the ability to handle immature UI systems without turning a feature-level skill into a design-system program.

#### 5. One-downstream-skill pilot does not eliminate the broader methodology impact

Opus sensibly recommends piloting with `team-impl-cc` first.

GPT agrees that this is the best implementation consumer to test first. But GPT would add that if the change is framed as an official new phase in the methodology, then the broader pipeline surfaces still need to be updated even before multi-consumer rollout. In particular, spec orchestration and public phase-order documentation still need to reflect the new phase shape.

### What GPT thinks Opus underemphasized

#### 1. The missing impact on `ls-team-spec`

GPT considers this the biggest omission in the proposal. The current proposal names downstream implementation consumers, but does not account for the fact that the spec-orchestration skill currently assumes a direct Tech Design -> Publish Epic transition. If the official pipeline changes, `ls-team-spec` must change too.

#### 2. The need for a hard invocation rubric

Both reviews agree that “optional” is too loose, but GPT places more weight on this than Opus does. A methodology like Liminal Spec works best when phase entry is crisp. “Frontend-heavy” or “customer-facing” is not enough. The proposal needs explicit triggers for when UI Design is required.

#### 3. The missing supporting analysis file

The proposal cites `docs/research/frontend-ui-design-skill-analysis.md`, but that file does not currently exist in the workspace. GPT views this as a small but real weakness in the review trail.

#### 4. The build and release surface area

GPT places more weight than Opus on the implementation cost beyond content authoring. A new skill changes:

- standalone pack ordering
- build mappings
- tests
- pipeline documentation
- release artifact expectations

That does not argue against the proposal, but it does raise the real cost of adopting it.

### Bottom line from GPT's comparison

The substantive overlap between the Opus review and the GPT review is high. Both reviews agree that:

- the problem is real
- the proposal is directionally strong
- the current draft is too heavy
- the verification model is not yet strong enough
- the Tech Design / UI Design boundary needs tightening
- the proposal should be narrowed or piloted before broad rollout

The main difference is strategic emphasis:

- **Opus** is more willing to move toward a smaller new skill now.
- **GPT** is more inclined to start with a Tech Design companion artifact first, then promote it to a phase only if the lighter intervention fails.

That is a real difference, but it is a difference of rollout strategy more than a difference in diagnosis.

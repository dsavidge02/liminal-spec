# GPT Review: `ls-ui-design` Proposal

**Review date:** 2026-04-16  
**Reviewer:** GitHub Copilot (GPT-5.4)

## Overall Assessment

The proposal identifies a real structural gap in the current Liminal Spec workflow. The existing pipeline is strong on behavioral specification, technical architecture, and traceability into implementation, but it does not currently provide a dedicated artifact for screen-level UI intent, state presentation, visual system reuse, or responsive behavior. That absence plausibly explains recurring frontend quality misses.

The proposal is thoughtful, mostly well-scoped, and materially stronger than a vague "add mockups" idea. It correctly reframes the problem as one of **structured UI specification**, not decorative design output.

That said, the proposal is **not yet fully operationally complete**. It makes a convincing case for a new artifact, but a less complete case for a new top-level skill, and it leaves several workflow and verification details underdefined.

My judgment is:

- The problem is real.
- The proposed placement is correct.
- The artifact concept is strong.
- The current v1 is too heavy for the repo's methodology norms.
- The downstream integration plan is incomplete.
- The verification model needs a more honest ceiling and a clearer evidence surface.
- The proposal should be tightened and piloted before implementation.

---

## Findings

### 1. The proposed v1 is too heavy and not yet aligned with the repo's "earn complexity" principle

The biggest point I underweighted in my original review is the size of the proposed v1 itself.

As written, the proposal front-loads a 16-section artifact. In this repo, that is not a neutral formatting choice. It implies a large amount of methodology surface has already been justified and stabilized. Given the project's own guidance to earn complexity rather than front-load it, I think the proposed v1 is currently overbuilt.

The strongest version of this idea is not "ship the full section inventory now and trim later." It is the opposite: prove a smaller v1 first, then add sections only when their absence causes real downstream errors.

This also changes how I view the **Establish** path. The inherit / extend / establish frame is still a strong analytical lens, but the full Establish mode is too ambitious for a feature-level v1. Defining foundational tokens, spacing scale, typography hierarchy, and reusable primitives in one pass starts to look like design-system work, not feature-level UI specification. For v1, I would either:

- defer Establish entirely, or
- narrow it to documenting minimum required additions and visible system gaps rather than attempting a full system definition

### 2. The proposal underestimates the workflow surface area

The document correctly identifies downstream changes needed in `ls-publish-epic`, `ls-team-impl-cc`, `ls-team-impl`, and `ls-codex-impl`.

That is not sufficient.

The current repo also has an explicit orchestration skill for the spec pipeline: `ls-team-spec`. That skill assumes the current sequence is:

`Epic -> Tech Design -> Publish Epic`

This assumption is embedded in both the process narrative and the concrete prompt structure. Adding a new UI Design phase changes the shape of the pipeline, so `ls-team-spec` would also need changes in at least these areas:

- pipeline entry logic
- phase transition rules
- reading journeys
- prompt maps
- human review checkpoints
- phase acceptance / logging structure

The public docs would also need coordinated updates, because the main README currently describes the pipeline as five build phases plus continuity, with `ls-tech-design` followed directly by `ls-publish-epic`.

This omission matters because it is the difference between a good concept and a well-integrated methodology change.

### 3. The verification model needs a more honest ceiling and a clearer evidence surface

The proposal is right that implementation verification must check the UI spec or the artifact becomes advisory.

But the current implementation lanes mostly verify by reading code, running gates, and checking spec compliance at the behavioral and architectural level. That works well for AC/TC fidelity and test integrity. It does **not** automatically produce good evidence for visual fidelity.

The missing piece is an evidence model.

If the problem is that the implementation can satisfy behavior while still looking wrong, then reviewers need something more concrete than prose plus code. Without that, "visual compliance" becomes taste-based judgment, which is weaker than the rest of Liminal Spec's confidence chain.

I would want the proposal to define at least one required verification surface when a UI spec exists, for example:

- fixed-breakpoint screenshots for named states
- Storybook or equivalent state renders
- explicit state capture checklist per screen
- responsive snapshots at agreed breakpoints
- accessibility verification artifacts tied to the UI spec

Until that exists, the proposal improves guidance, but does not fully solve enforceability.

I would also state the limitation more directly than I did in the first draft of this review: absent stronger runtime evidence, agents can verify **structural UI compliance** more reliably than they can verify visual polish, rhythm, or hierarchy. In that mode, visual quality remains at least partly a human gate. The methodology should be explicit about that rather than implying full visual verification from code review alone.

### 4. The proposal proves the need for a UI-spec artifact more clearly than it proves the need for a new top-level skill

This is the biggest strategic question in the document.

The proposal acknowledges the central divergence: whether to create a dedicated new skill or enhance `ls-tech-design`, especially through the existing companion-doc structure. The document resolves that in favor of a dedicated skill on the grounds that a separate gate makes the work happen explicitly.

That argument is understandable, but not fully decisive yet.

Today, `ls-tech-design` already supports multiple output documents and traceable companion docs. So the real question is not "do we need a ui-spec document?" The answer to that may well be yes. The real question is:

**Should that document be produced as part of Tech Design, or as a separate pipeline phase?**

Given that the repo recently simplified the pipeline by removing story-sharding and story-tech-enrichment ceremony, any re-expansion of the phase model needs a very high bar.

Right now, I think the proposal makes a strong case for one of these two paths:

1. Add a `ui-spec.md` artifact as a conditional tech-design companion first.
2. Promote it to a distinct phase only if that lighter-weight approach proves too easy to skip or too easy to blur into architecture work.

If you want to go straight to a top-level skill, the proposal should make the "why not tech-design companion?" case more rigorously.

I would now add a stronger version of Opus's pushback here: the proposal has not fully ruled out the possibility that the real failure is not structural insufficiency in Config B, but lack of enforcement. A focused spike on "UI-spec companion + one verifier change" is still the best low-cost discriminating test.

### 5. The boundaries and overlaps are directionally right, but still exposed to drift

The proposal does attempt to separate responsibilities:

- Epic owns behavior.
- Tech Design owns architecture and interfaces.
- UI Design owns visual contracts, screen/state mapping, and interaction presentation.

That is the correct direction.

The problem is that the proposal also says the UI spec includes component specifications with prop contracts, layout constraints, and state presentations. That wording is dangerously close to interface duplication.

In practice, if both documents can describe the same component in detail, one of two things will happen:

- the documents drift, or
- the author starts duplicating and syncing content manually

Neither outcome fits Liminal Spec's current discipline.

I would tighten the contract like this:

- **Tech Design owns**: component identity, module placement, TypeScript interfaces, data contracts, event/model boundaries, state management approach.
- **UI Spec owns**: visual structure, state presentation rules, layout behavior, responsive behavior, token usage, accessibility expectations, motion/interaction details.
- **UI Spec references Tech Design identifiers** rather than redefining them.

If this is not made very explicit, the feature will create a dual-source-of-truth problem.

I would also broaden this concern beyond Tech Design alone. Opus is right that there is still unresolved overlap with nearby audit surfaces, especially `ls-current-docs`. "Different lens" is not a full boundary. The proposal should say more clearly what a UI-specific inventory adds when recent current-state documentation already exists.

There is also a precision issue around the confidence chain. I agree with Opus that the UI spec should be described as **annotating** or **decorating** the frontend surface of ACs, not extending the core chain itself. The authoritative chain remains:

`AC -> TC -> Test -> Implementation`

The UI spec should trace ACs to screens and states, but it should not be described as if it becomes a new upstream authority parallel to ACs.

### 6. The optional invocation rule is too soft

The proposal says the skill is optional and should be invoked for frontend-heavy or customer-facing work.

That is probably true as a principle, but it is not precise enough as an operational gate.

Liminal Spec works best when phase entry/exit decisions are crisp. If you leave this one loose, teams will make inconsistent calls:

- some will skip it for screens that clearly need it
- some will invoke it for nearly every UI change
- some will use it only when they already have mockups
- some will treat it as a polishing pass instead of a requirements artifact

I would want a hard trigger rubric, such as invoking UI Design when any of these are true:

- the feature introduces net-new screens or major screen re-composition
- the feature introduces meaningful responsive behavior beyond trivial stack/reflow
- the feature adds or materially changes reusable UI primitives or token families
- the feature requires explicit handling of complex state matrices in the UI
- the user provides screenshots, mockups, or Figma exports that must be honored
- the feature's business value materially depends on user trust, clarity, or visual quality

Without something like that, the optional nature of the skill will weaken adoption discipline.

I do think Opus is right to be cautious here as well: the trigger rubric should not become its own large ceremony. The right answer is a small, explicit gate, not a secondary mini-methodology.

### 7. The proposal needs stronger grounding and a cleaner evidence trail

The proposal says these are recurring failures across multiple epics, but it does not name the incidents. I agree with Opus that this weakens the proposal more than I said in my first draft.

Methodology changes are easier to evaluate when they are tied to specific failures. If the proposal wants to justify a new artifact and possibly a new phase, it should cite at least two named cases where:

- frontend output met behavioral requirements but missed key UI quality expectations
- state coverage or responsive behavior was underspecified
- implementation agents inferred badly because screen-level intent was absent

Those cases then become the test for whether the proposed intervention actually helped.

The evidence trail also has a smaller but still real documentation problem.

The proposal cites an independent Copilot analysis file:

- `docs/research/frontend-ui-design-skill-analysis.md`

That file does not currently exist in the workspace.

This does not damage the logic of the proposal itself, but it does weaken the review trail. If the proposal is meant to stand as a durable design document, the referenced supporting artifact should either be added or the reference removed/rephrased.

---

## What Is Strong In The Proposal

### The core diagnosis is credible

The repo's current `ls-tech-design` phase is architecture-heavy and test-heavy. It explicitly covers system context, module architecture, interfaces, sequence flows, TC-to-test mapping, and chunking. It does **not** currently describe a comparable screen/spec discipline for frontend behavior and presentation.

So the proposal is not inventing a fake gap. It is naming a real one.

### The placement is correct if this becomes a distinct artifact

`Epic -> Tech Design -> UI Design -> Publish Epic -> Implementation`

This is the right location.

Running UI design before tech design would make it speculative. Running it after publish epic would be late enough to create churn. The proposal gets that sequencing right.

### The inherit / extend frame is excellent, but Establish is too ambitious for v1

This is still one of the best parts of the document.

The proposal does not assume greenfield creative freedom. It instead forces the artifact to answer a disciplined question:

- inherit an existing design system
- extend a partial one

That is a very good guardrail against both over-design and reinvention.

What I would revise after Opus's feedback is my earlier comfort with shipping all three paths in v1. The decision frame is strong, but the full Establish path is too big for a first pass at a feature-level skill. It should either be deferred or narrowed to documenting minimum required additions and visible system gaps.

### Treating reference materials as first-class inputs is the right call

Many real projects do not start from a blank slate. They start from screenshots, mockups, competitor references, design debt, or partial visual direction. The proposal's priority order handles that reality well.

The "honor or explicitly deviate with rationale" rule is especially good.

### The proposal correctly rejects "mockup" framing

This is another strong choice. A text-first AI workflow does not need an ornamental mockup stage. It needs a **binding UI specification artifact** that downstream consumers can use. The proposal mostly stays on that track.

---

## Key Risks If Implemented As Written

### Risk 1: It reintroduces pipeline ceremony without a lean enough v1 or a hard enough entry gate

The pipeline was recently simplified. Adding a new phase with a 16-section artifact and a soft "optional" trigger risks recreating the same kind of ceremony the repo intentionally removed.

### Risk 2: It becomes advisory instead of binding

If implementation and verification consume the UI spec only as extra reading, but there is no concrete evidence model, the artifact will improve outcomes inconsistently rather than structurally. In that mode, structural compliance may improve while visual quality still depends on human judgment.

### Risk 3: It creates overlapping ownership with Tech Design and adjacent audit surfaces

If the UI spec and Tech Design both talk about components too richly, or if the UI inventory duplicates nearby current-state audits, authors and reviewers will struggle to know which document wins.

### Risk 4: It is not yet grounded enough in named failures to justify its weight

The proposal argues from recurring pain, but does not yet tie the new artifact to specific incidents. That makes it harder to judge whether the proposed intervention is the right size.

### Risk 5: It expands implementation surface area more than the checklist currently admits

The checklist already understates the real cost because it omits spec orchestration, phase-order documentation, and build/release implications.

---

## Recommendation

I would not reject this proposal.

I also would not implement it exactly as written.

After incorporating the strongest Opus findings, my updated recommendation is:

### 1. Shrink v1 materially

Do not start with the full 16-section shape.

A better v1 is closer to 6-8 sections, covering only the core functions:

- Reference Material Analysis (conditional)
- Visual System Strategy
- Existing UI Inventory
- Screen Inventory
- AC-to-Screen/State Map
- State Coverage per Screen
- Component Specifications
- Open Questions and Assumptions

Supporting concerns like reuse notes, token impact, responsive rules, and interaction details can be folded into component-level sections initially and split out only if their absence proves costly.

### 2. Narrow or defer the Establish path

For v1, the safest posture is Inherit and Extend.

If a project has no meaningful UI system, the v1 artifact should document minimum required additions and visible gaps, not attempt to create a feature-level design system program by itself.

### 3. State the verification ceiling honestly and require a verification surface

The review should no longer imply that prose plus code review is enough for full visual verification.

The methodology should require a verification surface when a UI spec exists, but leave the exact form to the project. That surface might be screenshots, state captures, Storybook renders, accessibility evidence, or another project-appropriate artifact. The critical point is that the project must choose one.

Absent that, structural UI compliance is more realistically verifiable than visual polish, and visual quality remains at least partly a human gate.

### 4. Tighten the boundary and confidence-chain language

The UI spec should reference Tech Design identifiers rather than redefine them.

It should also be described precisely: the UI spec annotates the frontend surface of ACs. It does not replace ACs as the upstream authority, and it does not change the core chain of:

`AC -> TC -> Test -> Implementation`

### 5. Ground the proposal in evidence

Before implementation, the proposal should:

- cite at least two named failure incidents
- explain what the current workflow missed in those cases
- explain how the UI-spec artifact would have changed the outcome
- fix or remove the missing `frontend-ui-design-skill-analysis.md` reference

### 6. Choose the rollout path deliberately

My preferred low-risk path is still:

- run a focused Tech Design companion / Config B spike first
- test whether one verifier change plus a UI-spec companion closes most of the gap
- promote to a top-level phase only if that lighter intervention fails

If the owner wants the explicit gate immediately, I would still ship a lean v1 first, pilot one implementation consumer first, and update `ls-team-spec` plus the public phase-order docs if the official pipeline shape changes.

---

## Bottom Line

This is still a serious proposal, not a decorative add-on.

It identifies a real blind spot in the current methodology and proposes a credible artifact to close it. The strongest parts remain the diagnosis, the post-Tech-Design placement, the codebase-aware design inventory, the reuse-first design stance, and the insistence that UI intent should be explicit rather than inferred.

After incorporating Opus's feedback, I would now characterize the draft more sharply:

- the concept is strong
- the current v1 is overbuilt
- the verification promise needs more honest framing
- the methodology boundaries need tighter precision
- the proposal needs better incident grounding before it earns this much new ceremony

If those issues are addressed, this could become a worthwhile addition to the Liminal Spec workflow. If they are not, the risk is that it becomes a well-written but overweight artifact that improves some outcomes without reaching the enforceable rigor the rest of the methodology is aiming for.

---

## Revision Note

This review was revised after the Opus cross-review to incorporate several missed or underweighted points:

- section inflation and the repo's "earn complexity" principle
- the v1 scope problem around the Establish path
- the need to state the verification ceiling more explicitly
- precision around how the UI spec relates to the confidence chain
- the need for named failure incidents to justify the proposal's weight
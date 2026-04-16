# Plan: `ls-tech-design-v2` + `ls-team-impl-v2` — Experimental UI Companion Pilot

**Status:** Plan — pending owner approval
**Date:** 2026-04-16 (revised after GPT cross-review of plan)
**Sources synthesized:**
- `docs/research/ui-design-skill-proposal.md` (original proposal)
- `docs/research/ui-design-skill-opus-review.md` (Opus review + GPT cross-verification)
- `docs/research/ui-design-skill-gpt-review.md` (GPT review, revised after Opus cross-review)
- `docs/research/ls-tech-design-v2-plan-gpt-review.md` (GPT review of this plan; six findings verified against repo state and applied below)

---

## Goal

Close the structural UI specification gap in the Liminal Spec pipeline **without** committing to a new top-level skill or modifying the published phase model. Do this by shipping two experimental, parallel skills:

- `ls-tech-design-v2` — Tech Design plus a conditional `ui-spec.md` companion artifact.
- `ls-team-impl-v2` — Team implementation orchestration (Codex subagent path) that consumes the optional UI spec.

Both ship alongside the existing `ls-tech-design` and `ls-team-impl`. Neither v1 skill is modified. Users opt into the v2 pair when they want to test the UI companion path; existing workflows continue to use the v1 pair.

Run it as the experimental option until evidence justifies one of three outcomes:

1. **Promote** — graduate the UI spec capability to a dedicated `ls-ui-design` skill in its own pipeline phase.
2. **Merge** — fold the UI companion into `ls-tech-design` proper, retiring `ls-tech-design-v2`.
3. **Retire** — discard the experiment if the companion doesn't produce measurable improvement.

This path is the convergent recommendation of both reviews. It captures the artifact and the enforceability that the original proposal targets, while avoiding the costs of a new pipeline phase, an `ls-team-spec` overhaul, and conditional branches in four implementation skills.

---

## What the v2 Pair Is (and Isn't)

### Is

- A pair of **parallel** skills (`ls-tech-design-v2` + `ls-team-impl-v2`) that live alongside the existing `ls-tech-design` and `ls-team-impl`. Same inputs, same core outputs, plus the conditional UI companion artifact and its consumption.
- An **experiment** with explicit success criteria and a defined exit point.
- The **downstream consumer** is the parallel `ls-team-impl-v2`. The existing `ls-team-impl` is not edited, so users can flip back to the v1 pair at any time.
- **Backwards compatible.** Existing projects keep using `ls-tech-design` + `ls-team-impl`. New work that wants the UI companion opts into the v2 pair.

### Isn't

- Not a replacement for `ls-tech-design` or `ls-team-impl`. All four skills ship side-by-side until the experiment resolves.
- Not a new pipeline phase. The published phase model stays `Epic → Tech Design → Publish Epic → Implementation`.
- Not a coupling change to `ls-publish-epic`, `ls-team-impl-cc`, `ls-codex-impl`, or `ls-team-spec`. Those stay untouched until evidence justifies broader rollout.
- Not a 16-section artifact. The companion ships at 6–8 sections.
- Not a full design-system definition tool. The Establish path is narrowed to gap documentation only.

---

## Design Decisions Inherited from the Reviews

These are the synthesized decisions both reviewers converged on, applied to the v2 design.

### Section count: 6–8, not 16

The companion ships with these sections:

1. Reference Material Analysis (conditional — omitted if no references)
2. Visual System Strategy (Inherit / Extend only — Establish narrowed below)
3. Existing UI Inventory
4. Screen Inventory
5. AC-to-Screen/State Map
6. State Coverage per Screen
7. Component Specifications (folds in prop reuse notes, token usage, and layout cues per component)
8. Open Questions and Assumptions

Sections deferred until their absence causes observable downstream errors: Component Reuse Matrix, Proposed Reusable Additions (separate), Token and Theme Impact (separate), Layout and Responsive Rules (separate), Interaction and Motion Notes (separate), Visual Direction (separate), Issues Found.

### Establish path: narrowed, not deferred

If no meaningful UI system exists, the v2 companion **documents minimum required additions and visible system gaps**. It does not attempt to define a full design system (token taxonomy, type scale, component library). Full design-system establishment becomes a separate concern (potential future `ls-design-system` skill).

### Invocation rubric: three triggers, not six

The companion runs when **any** of these is true:

1. The feature introduces net-new screens or major screen re-composition.
2. The feature adds or materially changes reusable UI primitives or token families.
3. The user provides reference materials (mockups, screenshots, Figma exports) that must be honored.

Anything beyond this risks turning the gate into its own ceremony.

### One-way ownership contract

To prevent dual-source-of-truth drift between the tech design and the UI companion:

- **Tech design owns:** component identity, module placement, TypeScript interfaces, data contracts, event/model boundaries, state management approach.
- **UI companion owns:** visual structure, state presentation rules, layout behavior, responsive behavior, token usage, accessibility expectations, motion/interaction details.
- **UI companion references tech design identifiers** (component name, props symbol) rather than redefining them. If the companion is redefining an interface, it has crossed the line.

### Verification surface required, form project-chosen

When the companion is produced, the project must commit to **at least one** verification surface. The methodology does not mandate a tool. Candidate forms:

- Fixed-breakpoint screenshots for named states
- Storybook or equivalent state renders
- Snapshot tests at agreed breakpoints
- Accessibility verification artifacts
- Per-screen state capture checklist verified by human review

Without a committed verification surface, the companion stays advisory and the experiment fails its own test.

### Confidence chain framing

The companion **annotates** the frontend surface of ACs. It does not extend the chain. Authoritative chain remains `AC → TC → Test → Implementation`. The companion's traceability map is `AC → screen/state`.

---

## Repo Changes

All changes are additive. Nothing in the existing skill set is removed or renamed.

### 1. New phase source file

**Create:** `src/phases/tech-design-v2.md`

Contents derive from `src/phases/tech-design.md`. The v2 file adds:

- An invocation-rubric block at the top describing when the UI companion runs (the three triggers above).
- A new **Output Contract Addendum** section that explicitly classifies `ui-spec.md` as a *separate* artifact class, sibling to the Tech Design output:
  - Tech Design output stays Config A (2 docs) or Config B (4 docs). The "Never go 3" rule is preserved.
  - The UI companion, when invoked, adds **exactly one** artifact, `ui-spec.md`, alongside the Tech Design output. It is not counted toward the Tech Design configuration.
  - This is a v2-only addendum to the Tech Design contract; it does not modify v1 behavior.
- A new "UI Companion" section after the existing tech design content, describing:
  - The 8-section companion structure (sections listed above).
  - The Inherit / Extend / narrowed-Establish strategy gate, with the codebase-aware audit step.
  - Reference materials priority (when present, they are the starting constraint).
  - The one-way ownership contract with the rest of the tech design.
  - The verification surface requirement.
  - The companion output filename: `ui-spec.md` alongside `tech-design.md`.
- A boundary callout: when the companion is **not** invoked, v2 behaves identically to v1.

The body of the existing tech design content stays as-is. The companion is appended; nothing is rewritten. The Output Contract Addendum makes the relationship to the existing 2-doc / 4-doc rule explicit so the experiment doesn't silently violate it.

### 2. New manifest entries

**Edit:** `manifest.json` — add **two** entries:

```json
"ls-tech-design-v2": {
  "name": "ls-tech-design-v2",
  "description": "Experimental Tech Design with optional UI companion artifact for frontend-heavy work. Produces tech-design.md and conditionally ui-spec.md with codebase-aware visual specification, screen inventory, state coverage, and AC-to-UI traceability. Pilot skill — use ls-tech-design for stable workflow.",
  "phases": ["tech-design-v2"],
  "shared": [
    "confidence-chain",
    "dimensional-reasoning",
    "verification-model",
    "writing-style",
    "testing"
  ],
  "templates": ["tech-design.template"],
  "examples": ["tech-design-verification-prompt"]
}
```

```json
"ls-team-impl-v2": {
  "name": "ls-team-impl-v2",
  "description": "Experimental team implementation orchestration paired with ls-tech-design-v2. Opus teammates manage external CLI subagents (Codex or Copilot) for implementation and verification, with conditional handling of the optional ui-spec.md companion. Pilot skill — use ls-team-impl for stable workflow.",
  "phases": ["team-impl-v2"],
  "shared": []
}
```

The v2 tech design entry mirrors `ls-tech-design` exactly — same `shared` list, same `templates`, same `examples`. This is intentional (corrected after GPT review): the experiment must isolate the UI companion as the only meaningful variable. If the v2 entry dropped the existing template or verification-prompt example, a successful or failed outcome would be impossible to attribute cleanly to the UI companion versus the loss of the existing scaffolding.

`ls-team-impl-v2` mirrors `ls-team-impl`'s self-contained shape (no shared dependencies). It also stays a generic external-CLI orchestration skill (Codex or Copilot user-selectable) — not a Codex-specific rewrite. The pilot will *select Codex* when invoking v2 (see Pilot section), but the skill itself preserves the v1 CLI-selection behavior.

### 3. Build standalone name mapping

**Edit:** `scripts/build.ts`

The existing `STANDALONE_NAMES` map uses pipeline-ordered numeric prefixes (`03-technical-design`, `06-team-implementation`, etc.). Experimental skills must opt intentionally into this scheme, not drift around it. For the v2 pair, slot alongside their v1 counterparts using a `v2` suffix on the pipeline number:

```ts
"ls-tech-design-v2": "03v2-technical-design-v2",
"ls-team-impl-v2": "06v2-team-implementation-v2",
```

This produces `dist/standalone/03v2-technical-design-v2-skill.md` and `dist/standalone/06v2-team-implementation-v2-skill.md`. The `v2` in the numeric prefix keeps the experimental skills visually adjacent to their v1 originals when the standalone directory is sorted, while making the experimental status unmistakable. Corrected after GPT review — the original plan's bare `tech-design-v2-skill.md` did not match the existing pattern.

No new build logic is needed — the manifest-driven composition handles everything.

### 4. Build tests

**Edit:** `scripts/__tests__/build.test.ts`

Add assertions for:

- Expected skill list includes both `ls-tech-design-v2` and `ls-team-impl-v2`.
- `dist/skills/ls-tech-design-v2/SKILL.md` and `dist/skills/ls-team-impl-v2/SKILL.md` exist with the expected frontmatter.
- The composed `ls-tech-design-v2` content includes both the existing tech design body **and** the UI companion section.
- The composed `ls-team-impl-v2` content includes the existing team-impl body **and** the conditional UI spec artifact + handoff blocks.
- Standalone files `dist/standalone/tech-design-v2-skill.md` and `dist/standalone/team-impl-v2-skill.md` exist.
- `ls-tech-design` and `ls-team-impl` are still present and unchanged (regression guard).

No version assertion bump is needed for the experiment — version moves only when the experiment resolves.

### 5. CLAUDE.md

**Edit:** `CLAUDE.md`

Two changes:

1. Add **two** rows to the skill table marking `ls-tech-design-v2` and `ls-team-impl-v2` as **experimental**, with a short note pointing to this plan document. Do **not** modify the published pipeline diagram or phases list. The experiment stays out of the official pipeline narrative until promotion or merge.
2. Add a new short **Experimental Skill Convention** subsection under "PR Checklist" that records the policy exemption used here:

   > Skills marked experimental in the manifest (currently `ls-tech-design-v2`, `ls-team-impl-v2`) are exempt from the standard skill-addition documentation rules: they are listed in `CLAUDE.md` only, not in `README.md`, `src/README-pack.md`, or `src/README-markdown-pack.md`. Public docs are updated only at promotion or merge time, when the experiment resolves. This keeps the public surface stable while experiments run.

This makes the no-public-docs stance explicit policy rather than an undocumented exception. Corrected after GPT review — the prior plan implicitly violated the existing skill-addition rules in CLAUDE.md by skipping the README/pack-README updates without saying so.

### 6. README.md

No changes. The experiment is intentionally not advertised in the public README until the value question is resolved. README claims are durable; experiments are not.

### 7. Pack READMEs

No changes to `src/README-pack.md` or `src/README-markdown-pack.md`. Same reasoning as README.

### 8. Pilot downstream consumer (new parallel skill)

**Create:** `src/phases/team-impl-v2.md`

The pilot consumer is a new parallel skill, `ls-team-impl-v2` — Opus teammates managing external CLI subagents for implementation and verification, with the conditional UI spec consumption baked in. The existing `ls-team-impl` is **not edited**; users can flip back to the v1 pair at any time.

`ls-team-impl-v2` stays a **generic external-CLI orchestration skill** (Codex or Copilot), preserving the v1 CLI-selection step. It is *not* a Codex-only rewrite. Corrected after GPT review — the prior plan repeatedly described v2 as the "Codex subagent path," but inheriting the v1 body unchanged would not actually make it Codex-specific. To keep the experiment clean, the skill stays generic and the **pilot itself selects Codex** at the CLI-selection step. This isolates the UI companion as the only meaningful variable in the experiment.

The v2 file inherits its body from `src/phases/team-impl.md`. On top of that body, it adds the following conditional behavior when a `ui-spec.md` artifact is present alongside the tech design:

- **Artifact collection (On Load):** the UI spec is added as an optional artifact path the orchestrator collects.
- **Implementer handoff template:** the Codex implementer subagent's reading journey includes `ui-spec.md` after the tech design and before the epic. The handoff prompt instructs the implementer to honor the component contracts, state presentations, and (referenced) tech-design identifiers from the spec.
- **Reviewer handoff template:** the Codex reviewer subagent receives `ui-spec.md` as a verification target and checks:
  - Components named in the UI spec are present in the implementation.
  - State presentations specified in the UI spec are reachable in code (loading, empty, error, success, validation, etc.).
  - The committed verification surface from the spec is produced.
  - Tech-design identifiers referenced from the spec resolve cleanly (early signal that the one-way ownership contract is holding).

This is the structural verification ceiling honestly stated in the spec itself. Visual polish stays a human gate.

A short header at the top of `team-impl-v2.md` notes that the skill is experimental, paired with `ls-tech-design-v2`, and points to this plan document.

**Why a parallel skill instead of editing `ls-team-impl`?** Three reasons:

1. **Reversibility.** If the experiment retires, deleting the v2 source/manifest entries removes the experiment cleanly. No risk of leaving partial conditional logic behind in the stable v1 file.
2. **Concurrent A/B usability.** Owners can run the same kind of work through both pairs and compare outcomes directly. Editing v1 in place would force a flag-style toggle and muddy the comparison.
3. **Stable v1 surface.** `ls-team-impl` is the production-grade orchestration skill. The repo's "earn complexity" principle applies here too — the experimental conditional logic should not live in the stable skill until it has earned its place.

**Why pilot through `ls-team-impl-v2` (with Codex selected) instead of `ls-team-impl-cc`?** Three reasons:

1. The owner wants to exercise the external-CLI subagent orchestration path with Codex specifically.
2. External CLI subagents are more sensitive to prompt expansion and artifact list growth than Claude Code agents are. If v2's added artifact reading works cleanly when the orchestrator is driving Codex through the v2 skill, it will work in a `ls-team-impl-cc` equivalent essentially for free at promotion or merge time.
3. The Opus teammates orchestrating the CLI provide a useful intermediate layer for catching artifact-handoff problems early — the orchestrator can observe whether the reviewer subagent actually consumed the UI spec or silently ignored it.

**No changes** to `ls-team-impl`, `ls-team-impl-cc`, `ls-codex-impl`, `ls-publish-epic`, or `ls-team-spec` during the experiment. If the pilot proves out, those changes happen at promotion or merge time, not now.

---

## File-Level Summary

| File | Action | Reason |
|---|---|---|
| `src/phases/tech-design-v2.md` | Create | New phase source for v2 tech design |
| `src/phases/team-impl-v2.md` | Create | New phase source for v2 team implementation (Codex subagent orchestration with optional UI spec) |
| `manifest.json` | Edit (add two entries) | Wire both v2 skills into the build |
| `scripts/build.ts` | Edit (add two name mappings) | Standalone outputs for both v2 skills |
| `scripts/__tests__/build.test.ts` | Edit (add assertions) | Verify both v2 skills build and v1 skills unchanged |
| `CLAUDE.md` | Edit (two skill-table rows + new Experimental Skill Convention subsection) | Internal awareness + explicit doc exemption policy |
| `README.md` | No change | Don't advertise experiments |
| `src/README-pack.md` | No change | Same |
| `src/README-markdown-pack.md` | No change | Same |
| `src/phases/tech-design.md` | No change | v1 stays stable |
| `src/phases/team-impl.md` | No change | v1 stays stable |
| `src/phases/team-impl-cc.md` | No change | Defer until evidence |
| `src/phases/codex-impl.md` | No change | Defer until evidence |
| `src/phases/publish-epic.md` | No change | Defer until evidence |
| `src/phases/team-spec.md` | No change | Defer until evidence |

---

## Local Installation for Testing

Once the build passes, the v2 pair must be installed into the owner's local Claude skills directory so it can be invoked during the pilot. This is part of the build deliverable, not a separate manual follow-up.

**Install destination:** `~/.claude/skills/`

**Command (run from repo root after `bun run build`):**

```bash
cp -r dist/skills/ls-tech-design-v2 ~/.claude/skills/ls-tech-design-v2
cp -r dist/skills/ls-team-impl-v2 ~/.claude/skills/ls-team-impl-v2
```

This follows the same local-test pattern documented in CLAUDE.md ("Testing locally" → `cp -r dist/skills/ls-epic ~/.claude/skills/ls-epic`). After install, both skills become invokable as `ls-tech-design-v2` and `ls-team-impl-v2` in any Claude Code session.

**Re-install on every change.** Any edit to `src/phases/tech-design-v2.md` or `src/phases/team-impl-v2.md` requires `bun run build` followed by re-running the two `cp -r` commands above to refresh the installed copy. The installed `~/.claude/skills/<name>/SKILL.md` is a snapshot of the build output, not a live reference into the repo.

**Verification after install:** open a Claude Code session in the pilot project and confirm both skills appear in the available-skills list. Spot-check one section of each composed `SKILL.md` to confirm the install picked up the latest build, not a stale copy.

If the owner later wants to remove the experimental skills (for example, if Retire is the resolution), `rm -rf ~/.claude/skills/ls-tech-design-v2 ~/.claude/skills/ls-team-impl-v2` cleans up the local install. The repo-level removal is covered separately in Phase 4 of the implementation checklist.

---

## Success Criteria for Resolving the Experiment

The experiment resolves once **at least two** real frontend-heavy epics have run end-to-end with the v2 pair (`ls-tech-design-v2` + `ls-team-impl-v2`). Both pilots are on a greenfield project, so they exercise the narrowed Establish path and the reference-materials input, not the Inherit or Extend paths. Each pilot run produces a short retrospective covering:

- Did the UI companion catch presentation gaps the tech design alone would have missed?
- Did the Codex reviewer subagent actually check against the companion, or was it inert?
- Did Playwright screenshot capture produce signal worth its cost? Did the per-state coverage actually catch presentation problems the implementer would otherwise have missed?
- Did the one-way ownership contract hold, or did the companion start redefining interfaces?
- Did the invocation rubric trigger correctly?
- Did the narrowed Establish path produce something useful for a greenfield project, or did it underspecify because there was nothing to anchor against?
- Did the reference materials input do real work, or was it ceremonial?
- What did the companion fail to specify that mattered downstream?

### Evidence requirement

**Promote and Merge both require at least one brownfield run** (a project with existing UI patterns) in addition to the two greenfield runs. Two greenfield-only pilots are not enough evidence to justify either outcome — they only exercise the narrowed Establish path and the reference-materials input. The Inherit and Extend paths, which are likely the more common cases in mature codebases, would be entirely untested.

This is a correction after GPT review. The earlier draft allowed Promote or Merge from greenfield-only evidence, which would have overgeneralized from the narrowest path the experiment exercises.

### Decision matrix

| Outcome | Trigger | Action |
|---|---|---|
| **Promote** | Companion provably closes the gap *and* the explicit gate matters (teams skip it when offered as a TD section but use it when invoked as its own thing). **Requires both greenfield pilots and at least one brownfield run.** | Graduate to `ls-ui-design` skill in its own phase. At that point, do the full proposal: update `ls-team-spec`, README pipeline tables, pack READMEs, and the other three implementation skills. |
| **Merge** | Companion provably closes the gap *and* projects naturally produce it inside the tech design without being prompted. **Requires both greenfield pilots and at least one brownfield run.** | Fold the companion into `ls-tech-design` proper, retire `ls-tech-design-v2`. |
| **Partial validation — extend experiment** | Greenfield runs succeed but no brownfield run has happened yet. | Hold both v2 skills in place. Run a brownfield pilot when the next eligible feature comes up. Do not promote or merge yet. |
| **Retire** | Companion does not measurably improve outcomes after the greenfield pilots, or the verifier surface proves too costly relative to the lift. (Brownfield evidence not required for retirement — a clearly negative greenfield result is sufficient to discard the experiment.) | Remove `ls-tech-design-v2` and `ls-team-impl-v2`, document the negative result. The structural gap remains open and may need a different intervention. |

The experiment should not stay open indefinitely. **Set a soft cap:** if the experiment hasn't resolved within four pilot runs (greenfield + brownfield combined), force a decision based on whatever evidence exists.

---

## Open Questions — Resolutions

The four open questions raised in earlier drafts of this plan have been resolved. Recorded here so the rationale is durable.

### 1. Named failure cases — *resolved: not needed*

The original reviews asked for two named historical failures so the experiment had a baseline. Resolution: the pilot will run on a **new project**, so there is no historical case to anchor against. Evaluation will instead be forward-looking — does the v2 pair produce visibly better frontend outcomes on the pilot project than the v1 pair would have on the same project? See "Success Criteria" below; the retrospectives are the evidence, not historical incidents.

### 2. Missing referenced analysis doc — *resolved: remove the reference*

The original proposal (`docs/research/ui-design-skill-proposal.md`) cites `docs/research/frontend-ui-design-skill-analysis.md`, which doesn't exist. Resolution: **edit the proposal to remove that reference** before treating the proposal as a durable design document. (This is a small documentation cleanup, not a v2 build dependency — but worth doing now so the review trail is clean.)

### 3. Verification surface — *resolved with elaboration*

The v2 companion requires *that* a verification surface exists; the form is project-chosen. For the pilot, the project owner picks one of the options below before the first run. The recommendation for a greenfield pilot is at the bottom.

| Option | What it is | Cost | Signal strength | Best for |
|---|---|---|---|---|
| **State capture checklist** | A markdown checklist per screen listing every named state from the UI spec (loading, empty, error, success, validation, etc.). The reviewer confirms each state is reachable in code and produces the expected DOM. | Lowest. Pure documentation. | Weak. Confirms structural presence; cannot catch visual problems. | Smoke test for whether the spec's state coverage actually shows up in code. Almost free. |
| **Screenshot capture** | Use Playwright (or equivalent) to render the app at named states and breakpoints, save PNGs to a fixtures directory, human reviews the set. | Medium. Needs a render harness, a way to drive the app into each state, and a place to store images. | Strong for static layout, spacing, and visual coverage. Weak for interaction polish. | A greenfield project that does not already have a component playground. Smallest jump from "no verification" to "real visual evidence." |
| **Storybook (or equivalent component playground)** | Stand up Storybook with stories for each component/state in the UI spec. Each state is independently renderable and inspectable. | High. New tooling, ongoing maintenance, story authoring discipline. | Strongest. Every state is independently exercisable; reviewers and humans can both inspect the same surface. | A project that already has Storybook or has design-system intent that justifies the investment. Overkill for a short-lived pilot. |
| **DOM snapshot tests** | Jest/Vitest snapshot tests captured at named states. | Medium. Needs test setup but not a render harness. | Catches structural drift; misses visual problems entirely. | Projects with strong unit-test discipline that want regression protection without screenshot infrastructure. |
| **Hybrid** | State capture checklist *plus* one rendering surface (typically screenshot capture) for the highest-value screens only. | Medium-low. | Strong for the screens that matter; cheap for the rest. | Most pilots. Captures the upside of visual evidence without paying the full Storybook tax. |

**Decision: Playwright screenshot capture for every named state.** Not the Hybrid. Not "top screens only."

Rationale:

- Once the Playwright harness exists, marginal cost per state is low — each new state is one more test case, not a new infrastructure decision.
- The screenshot *is* the evidence that the state is reachable. A separate state-capture checklist becomes redundant bookkeeping over the same fact.
- On a greenfield project, components can be designed for state-reachability from the start (props that force loading / error / empty), which makes driving the app into each named state cheap.
- Screenshots do double duty: verification evidence now + visual regression baseline for future work. The checklist produces nothing reusable.
- "Top screens only" pushes a cut-line decision onto the reviewer at every spec, which is its own ongoing tax. Capturing all named states removes that decision.

**Pre-pilot setup:** budget a small Playwright spike before the first pilot's implementation phase begins. The spike covers: Playwright installed and runnable, a screenshots fixture directory, a baseline pattern for driving components into named states (force props, mocked responses), and a convention for storing/reviewing screenshots. The Codex reviewer subagent then expects screenshots at known paths as part of its verification step.

**Fallback (only if Playwright setup is blocked):** state capture checklist as a per-screen markdown artifact. Documented here only so the experiment is not blocked by tooling friction; the checklist is materially weaker signal and should not be the default.

### 4. Pilot project candidates — *resolved: greenfield project*

The pilot will run on a **new project** with no established UI patterns. This has direct implications for the experiment design that should be made explicit:

- **The Inherit path is not exercised.** No existing design system to inherit from.
- **The Extend path is not exercised.** No partial system to extend.
- **The narrowed Establish path is the only code path tested.** This means the pilot is also implicitly testing whether the narrowed Establish guidance ("document minimum required additions and visible system gaps") produces something useful when there is *nothing* yet, not just gaps in something partial.
- **Reference materials become especially important.** With no codebase signal to anchor on, any mockups, sketches, screenshots, or competitor references the owner provides become the dominant input. The companion's Reference Material Analysis section will carry more weight than it would on a brownfield project.
- **The first feature's UI spec doubles as the seed of the project's design system.** Whatever tokens, layouts, and components the v2 companion specifies for the first frontend-heavy epic become the de facto starting set the next epic inherits. The narrowed Establish path needs to be honest about this — it's documenting gaps in something that doesn't exist yet, which means it's effectively defining the starter set.

This is a useful test of the experiment but not a complete one. The v2 pair will need a brownfield run before it can claim to handle Inherit and Extend cases — that should be flagged as a limitation in whichever retrospective resolves the experiment, even if the greenfield results are positive.

---

## Implementation Checklist

Phase 1 — Pre-build:

- [ ] Owner confirms the experimental scope (this document).
- [ ] Edit `docs/research/ui-design-skill-proposal.md` to remove the dangling reference to `frontend-ui-design-skill-analysis.md`.
- [ ] Confirm pilot project's Playwright status. If not installed, run a small Playwright spike before the first pilot's implementation phase: install, fixture directory, state-driving conventions, and screenshot path conventions the Codex reviewer can expect.
- [ ] Owner confirms the pilot project (greenfield) and gathers any reference materials (mockups, sketches, competitor screenshots) that should seed the first run.

Phase 2 — Build:

- [ ] Create `src/phases/tech-design-v2.md` with the inherited body + UI companion section (~8 sections, narrowed Establish, one-way contract, verification surface requirement, three-trigger rubric).
- [ ] Create `src/phases/team-impl-v2.md` with the inherited team-impl body + conditional UI spec artifact collection, implementer handoff, and reviewer verification blocks (Codex subagent path).
- [ ] Add `ls-tech-design-v2` and `ls-team-impl-v2` entries to `manifest.json`.
- [ ] Add standalone name mappings for both new skills in `scripts/build.ts`.
- [ ] Add assertions for both new skills (and regression guards for v1) to `scripts/__tests__/build.test.ts`.
- [ ] Add two experimental rows to `CLAUDE.md`'s skill table with pointers to this plan, **and** add the new "Experimental Skill Convention" subsection that records the README/pack-README documentation exemption.
- [ ] Run `bun run verify` (build + validate + tests).
- [ ] Spot-check `dist/skills/ls-tech-design-v2/SKILL.md` and `dist/skills/ls-team-impl-v2/SKILL.md` for composition coherence.
- [ ] Spot-check `dist/standalone/03v2-technical-design-v2-skill.md` and `dist/standalone/06v2-team-implementation-v2-skill.md` for standalone usability.
- [ ] **Install the v2 pair into `~/.claude/skills/`** by running `cp -r dist/skills/ls-tech-design-v2 ~/.claude/skills/ls-tech-design-v2 && cp -r dist/skills/ls-team-impl-v2 ~/.claude/skills/ls-team-impl-v2`. Confirm both appear as available skills in a fresh Claude Code session.

Phase 3 — Pilot:

- [ ] Run pilot 1 on a frontend-heavy epic. Capture retrospective.
- [ ] Run pilot 2 on a different frontend-heavy epic. Capture retrospective.
- [ ] Owner reviews retrospectives against the decision matrix.

Phase 4 — Resolve:

- [ ] Promote, merge, or retire per the decision matrix.
- [ ] If promoting: do the full downstream work the original proposal listed (`ls-team-spec`, README, pack READMEs, other implementation skills, version bump).
- [ ] If merging: fold the companion section into `src/phases/tech-design.md`, fold the conditional UI spec consumption into `src/phases/team-impl.md`, remove both v2 source files and manifest entries, update tests, version bump.
- [ ] If retiring: remove both v2 source files and manifest entries, remove standalone name mappings and test assertions, **and** remove the local installs with `rm -rf ~/.claude/skills/ls-tech-design-v2 ~/.claude/skills/ls-team-impl-v2`. Document the negative result in `docs/research/`. The stable v1 skills are untouched.

---

## GPT Cross-Review Findings — Resolution Summary

A second-pass review by GitHub Copilot (`docs/research/ls-tech-design-v2-plan-gpt-review.md`) raised six findings against this plan. All six were verified against the actual repo state and applied above. Summary:

| # | Finding | Verified? | Resolution |
|---|---|---|---|
| 1 | Plan implicitly violates Tech Design "Never go 3" rule by appending `ui-spec.md` to Config A | Yes (`src/phases/tech-design.md` lines 11-30 explicitly state the 2-doc / 4-doc rule) | Added explicit Output Contract Addendum to v2 source: `ui-spec.md` is a separate artifact class, not counted in Tech Design configurations. |
| 2 | v2 manifest entry drops `templates` and `examples` from v1 | Yes (`manifest.json` shows v1 has both) | Updated proposed manifest entry to mirror v1 exactly. Notes that the experiment must isolate the UI companion as the only meaningful variable. |
| 3 | `ls-team-impl-v2` cannot be Codex-specific while inheriting v1 body unchanged | Yes (`src/phases/team-impl.md` line 77 explicitly asks the user to choose Codex or Copilot) | v2 stays generic CLI. Pilot itself selects Codex at the CLI-selection step. Plan reframed throughout to drop "Codex-specific" framing of the skill. |
| 4 | "No README / pack-README updates" violates skill-addition rules in CLAUDE.md | Yes (CLAUDE.md PR Checklist requires those updates for skill additions) | Added explicit Experimental Skill Convention to CLAUDE.md as part of the build, recording the exemption as repo policy rather than an undocumented exception. |
| 5 | Two greenfield runs are insufficient to justify Promote or Merge | Yes (the plan itself acknowledged Inherit/Extend would be untested) | Decision matrix updated: Promote and Merge require greenfield + at least one brownfield run. Added a Partial Validation outcome that holds the experiment open until brownfield evidence exists. Retire still works on greenfield-only evidence. |
| 6 | Standalone naming (`tech-design-v2-skill.md`) does not match existing pattern | Yes (`scripts/build.ts` lines 60-71 use `00-prd`, `03-technical-design`, etc.) | Standalone names changed to `03v2-technical-design-v2` and `06v2-team-implementation-v2`, slotting the experimental skills adjacent to their v1 counterparts. |

The other GPT open questions resolve cleanly from these fixes: (1) the experiment ships through the normal build but is exempt from public docs by explicit policy; (2) the experiment isolates the UI companion as the variable, with v2 skills mirroring v1 shape exactly; (3) `ui-spec.md` is an auxiliary artifact, not a formal Tech Design output, per the new addendum.

---

## Why This Plan Is the Right Shape

Both reviews independently arrived at "pilot the artifact as a Tech Design companion before promoting it to a phase." The original proposal is a strong long-term destination, but it asks for a lot of structural change up front:

- A new pipeline phase
- A 16-section artifact
- Conditional branches in four implementation skills
- Updates to `ls-team-spec`, README pipeline tables, and pack READMEs
- A full Establish path that approaches design-system work

`ls-tech-design-v2` lets us test whether the **artifact** produces the value the proposal claims, with a small fraction of the structural cost. If it works, the heavy proposal earns its weight on real evidence. If it doesn't, we learn that without having paid the full integration tax.

The cost of running the experiment is two new source files, two manifest entries, two standalone name mappings, and a few test assertions. The stable v1 skills (`ls-tech-design`, `ls-team-impl`) are untouched, which means the experiment is fully reversible: if it retires, deleting the v2 entries removes it cleanly. The upside is an honest answer to the strategic question both reviews flagged: **does the gap need a new top-level skill, a Tech Design companion, or something else entirely?**

Right now no one knows. After two pilots, we will.

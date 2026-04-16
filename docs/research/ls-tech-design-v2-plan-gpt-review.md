# GPT Review: `ls-tech-design-v2` + `ls-team-impl-v2` Plan

**Status:** Independent review — for cross-verification  
**Date:** 2026-04-16  
**Reviewer:** GitHub Copilot (GPT-5.4)  
**Subject:** `docs/research/ls-tech-design-v2-plan.md`

---

## Summary

The plan is directionally strong. It is trying to test the UI-spec companion path without paying the full cost of introducing a new top-level pipeline phase, and that is the right strategic instinct.

The main problem is not the experimental strategy. The main problem is that the plan describes the experiment as largely additive and low-friction, but several parts of that story do not hold against the current repo contracts.

In particular:

- `ls-tech-design-v2` does not cleanly fit the current Tech Design output contract if it simply appends a new `ui-spec.md` artifact.
- The proposed `ls-tech-design-v2` manifest entry does not actually mirror the current `ls-tech-design` skill, so the experiment changes more than the UI companion.
- `ls-team-impl-v2` cannot honestly be described as a Codex-specific pilot if it inherits the current `ls-team-impl` body unchanged.
- The documentation plan is inconsistent with the repo's current skill-addition rules if these skills are shipped through the normal manifest-driven build.

So the core idea is sound, but the implementation plan needs tightening before it is ready to build from.

---

## Findings

### 1. `ls-tech-design-v2` is internally contradictory on output structure

This is the most concrete structural problem in the plan.

The plan says:

- create `src/phases/tech-design-v2.md`
- keep the existing `src/phases/tech-design.md` body intact
- append a conditional `ui-spec.md` companion
- do not rewrite the core Tech Design content

That clashes directly with the current Tech Design contract in `src/phases/tech-design.md`, which says Tech Design always produces either:

- **Config A:** 2 docs
- **Config B:** 4 docs
- **Never go 3**

If `ls-tech-design-v2` keeps the v1 body intact and adds `ui-spec.md`, then Config A becomes 3 docs (`tech-design.md`, `test-plan.md`, `ui-spec.md`). That violates the inherited contract immediately.

So the plan has to choose one of these and say it explicitly:

1. `ls-tech-design-v2` rewrites the output contract rather than preserving the v1 body as-is.
2. `ui-spec.md` is not part of the formal output count and is treated as a different class of artifact.
3. The v2 skill is not a near-copy of v1 and should stop being described that way.

As written, it is trying to have both: "same body" and "new companion artifact". That does not survive contact with the current skill contract.

### 2. The proposed `manifest.json` entry does not actually mirror `ls-tech-design`

The plan says the new `ls-tech-design-v2` entry mostly matches `ls-tech-design`, with dimensional reasoning added.

That is not true against the current manifest.

The existing `ls-tech-design` entry already includes:

- `confidence-chain`
- `dimensional-reasoning`
- `verification-model`
- `writing-style`
- `testing`
- `templates: ["tech-design.template"]`
- `examples: ["tech-design-verification-prompt"]`

The proposed v2 entry drops the template and verification prompt.

Because the build system composes templates and examples directly from manifest metadata, this means the v2 skill would not actually be "v1 plus UI companion." It would be a different tech-design skill with less built-in content.

That matters because it changes the experimental variable. If the experiment succeeds or fails, you will not know whether the UI companion caused the outcome, or whether the loss of the existing template/example surfaces changed behavior.

If the goal is to isolate the UI-spec companion, the v2 manifest should preserve the v1 template and example composition unless the omission is deliberate and defended.

### 3. `ls-team-impl-v2` cannot be described as Codex-specific if it inherits the current `ls-team-impl` body unchanged

The plan repeatedly frames `ls-team-impl-v2` as the Codex orchestration pilot. It says:

- this is the Codex subagent path
- the owner specifically wants to test Codex orchestration
- the v2 skill inherits the body of `src/phases/team-impl.md`

The problem is that the current stable `ls-team-impl` skill is not Codex-specific. On load, it explicitly asks the user to choose **Codex or Copilot**.

So if `team-impl-v2.md` באמת inherits the v1 body and only adds UI-spec handling, then the experiment is not actually Codex-specific. It is still a generic external-CLI orchestration skill with an optional Codex choice.

That is not just wording drift. It changes what the experiment is testing.

The plan should choose one:

1. Make `ls-team-impl-v2` truly Codex-specific by rewriting the CLI-selection part of the inherited skill.
2. Keep it generic and stop describing it as a Codex-specific pilot.

Right now it says both at once.

### 4. The documentation strategy conflicts with the repo's own skill-addition rules if these skills ship through the normal build

The plan says to:

- add two new skills to `manifest.json`
- add them to the build
- add standalone names
- add build tests
- add rows in `CLAUDE.md`
- make **no changes** to `README.md`
- make **no changes** to `src/README-pack.md`
- make **no changes** to `src/README-markdown-pack.md`

That is in tension with the repo's current documented rule for adding skills. `CLAUDE.md` explicitly says that for skill additions or removals you update:

- `manifest.json`
- `scripts/build.ts`
- `scripts/__tests__/build.test.ts`
- `CLAUDE.md`
- `README.md`
- `src/README-pack.md`
- `src/README-markdown-pack.md`

If these experimental skills are being shipped through the normal manifest-driven build, then they are user-visible build artifacts and the current docs policy says they should be documented.

If the owner wants to exempt experiments from README and pack-README updates, that exemption needs to be stated explicitly as a repo-level policy change or this plan needs to define a non-release or gated-build path. Otherwise the plan is asking contributors to violate the repo's current documented process.

### 5. The success criteria allow promotion or merge before the experiment has tested the likely real-world cases

The plan says the experiment resolves after at least two real frontend-heavy epics, and both pilot runs are greenfield.

Later, the plan correctly acknowledges the consequence: those runs only exercise the narrowed Establish path, not Inherit or Extend, and a brownfield run is still needed before the v2 pair can claim to handle those cases.

That is the right limitation to name, but it conflicts with the decision matrix.

As written, the experiment can still:

- **Promote** to a top-level `ls-ui-design` skill, or
- **Merge** into `ls-tech-design`

after two greenfield runs.

That is too early. If greenfield is all you have exercised, then the experiment has only validated one branch of the intended strategy. It has not validated the branches that are probably more common in mature codebases.

At minimum, promotion or merge should require:

- the two greenfield pilots, and
- at least one brownfield run that exercises Inherit or Extend

Otherwise the plan risks overgeneralizing from the narrowest path.

### 6. The standalone filename plan does not actually follow the current output pattern

The plan proposes standalone names like:

- `tech-design-v2-skill.md`
- `team-impl-v2-skill.md`

and says this follows the same pattern as the existing mapping.

It does not.

The current standalone outputs use ordered numeric prefixes:

- `03-technical-design-skill.md`
- `06-team-implementation-skill.md`
- `06cc-team-implementation-claude-code-skill.md`

That numbering is part of the release surface. The plan's suggested names are not wrong in a technical sense, but they are not "the same pattern" as the current entries.

This matters because it affects how experimental artifacts appear in `dist/standalone` and whether users can meaningfully interpret their place in the sequence.

The plan needs to decide intentionally whether experimental skills:

- join the existing numbering scheme, or
- use a clearly separate experimental naming convention

Right now it gestures at the first while effectively doing the second.

---

## Open Questions / Assumptions

### 1. Is this experiment expected to ship in the normal release artifacts?

This is the most important unanswered operational question.

If yes, then the no-README / no-pack-README stance is inconsistent with current repo rules.

If no, then the plan should say so explicitly and explain how the build/release path keeps the experimental skills out of normal distribution.

### 2. Is the experiment trying to isolate the UI companion as the only meaningful variable?

If yes, then the v2 skills need to preserve more of the current v1 shape than the plan currently does — especially template/example composition and the generic-vs-Codex behavior of `ls-team-impl`.

If no, and the real experiment is "UI companion + Codex-specific orchestration + reduced v1 baggage," the plan should say that directly.

### 3. Is `ui-spec.md` a formal Tech Design output or an auxiliary artifact?

This needs a precise answer because it determines whether the v2 plan is overriding the current 2-doc / 4-doc Tech Design contract or not.

---

## Recommendation

I would treat this as a promising plan that needs one more tightening pass before implementation.

The shortest path to make it buildable is:

1. Explicitly rewrite the `ls-tech-design-v2` output contract so it no longer pretends to preserve the current 2-doc / 4-doc contract unchanged.
2. Decide whether `ls-tech-design-v2` preserves the current template/example composition. If not, say that the experiment is testing more than just the UI companion.
3. Decide whether `ls-team-impl-v2` is truly Codex-only or still generic CLI. Then align the inherited body claim to that choice.
4. Decide whether these skills are release-visible. Then align the README / pack-README plan to that answer.
5. Raise the evidence bar for Promote and Merge so greenfield-only runs cannot resolve the entire experiment.
6. Make standalone naming intentional rather than hand-wavy.

If those issues are fixed, the overall experimental shape looks sound.

---

## Bottom Line

The plan is good at the strategy level and weak in a few very concrete implementation contracts.

That is a fixable problem.

What it should not do is proceed under the assumption that these are minor wording issues. They are the exact points where an additive experiment becomes a different skill family than the plan says it is creating.

Claude should cross-check especially on these questions:

1. Is the `Never go 3` contradiction real, or is there a better way to classify `ui-spec.md`?
2. Should the experimental manifest entry preserve the v1 template/example composition to keep the experiment clean?
3. Should `ls-team-impl-v2` remain generic CLI, or be rewritten into a Codex-only skill?
4. Can the repo justify shipping manifest-backed experimental skills without README and pack-README updates, or does that need a separate policy exception?
5. Is two greenfield pilots enough evidence for Promote or Merge, given the plan's own admission that brownfield modes remain untested?
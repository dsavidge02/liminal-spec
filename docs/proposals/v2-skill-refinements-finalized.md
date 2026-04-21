# v2 Skill Refinements — Finalized Proposal

**Status:** Final — single proposal of record for the next v2 hardening pass
**Date:** 2026-04-19
**Scope:** v2 only. Stable v1 skills (`tech-design.md`, `team-impl.md`, `team-impl-cc.md`) remain unchanged.

This document is the consolidated and corrected output of the proposal/review cycle that ran on 2026-04-19. All sibling proposal and review files have been retired in favor of this one. Inputs that survive as durable references:

- `docs/findings/skill-refinements.md` — 8 cross-project refinements distilled from the streaming-control-panel pilot (Epic 1, Stories 0–9)
- `docs/findings/epic-1-files/` — pilot artifacts (`ui-spec.md`, 17 Playwright `__screenshots__/`, 10 story files, `decisions-log.md`)
- `docs/research/ls-tech-design-v2-plan.md` — original v2 experiment plan, decision matrix, exit criteria

---

## Goal

**The next v2 pilot should fail or succeed for substantive reasons — not because of silent delivery failures, ambiguous worker lifecycle, empty prompt pipes, invalid config snippets, or screenshot-default truncation.**

That single sentence justifies bundling all eight refinements. Some are UI-companion-specific (the experiment's actual subject); others are orchestration-harness hardening (which looks like cleanup but isn't). They ship together because before the v2 pilot can produce evidence the decision matrix can act on (Promote / Merge / Retire per the v2 plan), the pilot path itself has to be trustworthy.

This is a tightening pass, not a redesign. **Three contract-level changes are admitted explicitly** (the earlier framing of "exactly one" was incorrect — see Cross-cutting decisions for placement detail):

1. **Artifact shape (Refinement 8c).** `ui-spec.md` gains a required trailing Verification Surface section after Section 8.
2. **Acceptance-evidence trustworthiness (Refinement 6).** A CLI session is not trustworthy verification evidence unless the delivery-assertion marker is present in the recorded session output. A successful-looking transcript with no marker is empty-prompt fabrication, not verification — this sharpens what Control Contract invariants 1 and 3 already require.
3. **Acceptance-path observed-run requirement (Refinement 7b).** When the tech design prescribes an observed-run gate for runtime-path defects, story or epic acceptance requires the gate's commands to have been run and observed. Structural-gate passes alone do not satisfy acceptance.

The one-way ownership rule, the invocation rubric, the verification ceiling, and the existing three Control Contract invariants are otherwise unchanged. Refinements 6 and 7b extend invariants 1 and 3 with two short addenda; they do not rewrite them. Refinement 8c is the only artifact-template change.

---

## Why v2-only for now

Refinements 1–7b would also improve v1, but they stay in v2 only for now, for three reasons:

1. **Experimental containment.** The v2 pair is the active pilot surface. Tightening it first keeps the stable workflow unchanged while the experiment is still resolving.
2. **Better attribution.** If the next pilot improves, the gain is attributable to a specific hardened experimental package rather than a repo-wide diffuse set of edits.
3. **Cleaner promote/merge/retire decision later.** If these hardenings prove load-bearing, they merge into v1 deliberately at the experiment's resolution point, not by assumption.

The right framing is not "UI fixes plus orchestration cleanup." It is **one experimental hardening package for the v2 pilot path.**

---

## Pilot evidence corroboration

A second pass through `docs/findings/epic-1-files/` confirms the refinements are grounded in observed reality, not just retrospective interpretation. Three signals:

### Signal 1 — The produced `ui-spec.md` matched the 8-section template, plus a trailing Verification Surface section

The pilot author wrote sections 1–8 verbatim per the v2 template, then added a separate **Verification Surface** section after Section 8 codifying the Playwright state matrix (17 screenshots, palette × state coverage choices, the dev-only `?forceState=`/`?palette=` query-string driving mechanism, and the explicit "no fallback used" disposition). The v2 skill body already requires a verification surface to be specified — but the artifact template doesn't tell the author *where* in the spec to put it. The pilot author resolved this by appending it. **Refinement 8c (below) formalizes the resolution provisionally and admits it is a contract adjustment.**

### Signal 2 — The 17 screenshots corroborate the fix shape

The `__screenshots__/` directory contains all 17 baselines named in the spec's state matrix. Inspecting two captures:

- `landing-default-amber.png` — full-page composition: marquee, nav, hero, system status, error registry, palette switcher card, capability grid all present.
- `landing-default-amber-responsive-960x600.png` — explicitly tall (≈1300px) for a 960×600 viewport state. Unambiguously full-page capture.

These artifacts are post-fix. They **corroborate the shape of the fix** — that the final baselines ended up full-page — but they do not on their own prove the original truncation defect. The direct evidence for the original defect lives in `docs/findings/skill-refinements.md` (the human visual-review gate caught a clipped footer on the default-viewport baselines). The screenshot directory plus the findings doc together show both the failure and the recovery; neither alone is the full evidence chain.

### Signal 3 — `decisions-log.md` documents three additional Refinement 7 incidents, splitting cleanly into two defect classes

The findings doc cites three Story 7 incidents. `stories/decisions-log.md` documents three more from Story 8 packaging research, and the split between them is methodologically important:

*Config-shape defects (support Refinement 7a — authoring annotation step):*

- **Missing `node-linker=hoisted` in `.npmrc`** — symlinked pnpm layout works for dev and `pnpm start`; packaging is the first consumer that breaks.
- **Renderer `outDir` with `..`-relative path silently fails on Windows under Vite 8** — stdout claims success, disk shows nothing.

*Runtime-path defect (supports Refinement 7b — observed-run gate):*

- **Packaged ESM Electron 41 main with TLA chain in `app.whenReady()` hangs indefinitely** — dev mode unaffected because dev didn't use the same TLA sequence.

The third incident does **not** strengthen the case for snippet validation. The snippet itself is fine; the runtime path it triggers behaves differently from the dev-mode path. That is observed-run-gate evidence, not config-snippet evidence. Keeping these separate matters because the two defect classes have different fixes — 7a tightens the spec at authoring time; 7b enforces a gate at acceptance time.

Five concrete config-shape failure cases now back Refinement 7a; one runtime-path failure case backs Refinement 7b.

---

## How to read this document

For each refinement, this proposal documents:

1. **Pilot evidence** — what was observed (with artifact references)
2. **Why it matters** — the underlying failure mode the refinement closes
3. **Where it lands** — exact section(s) in v2 source, with approximate line numbers
4. **Proposed change** — near-final insertable text. Where source already covers part of the refinement, the proposal calls out only the delta.

Operational defaults (template materialization, message-validity altitude, marker shape, etc.) are resolved in the **Resolved Defaults** section near the end. Items considered and rejected from earlier drafts are listed in **Items Explicitly Deferred or Rejected**. Both sections exist so the implementation pass does not have to make methodology decisions on the fly.

---

# Group A — `ls-team-impl-v2` orchestration hardening

Six refinements landing in `src/phases/team-impl-v2.md`.

## Refinement 1 (A1) — Spawn-then-deliver for team-spawned teammates

**Pilot evidence.** First teammate spawn of the run emitted 4 idle notifications in 11 seconds with no substantive message. The spawn-time `prompt` argument did not reach the teammate's mailbox; the inbox was empty.

**Why it matters.** Every teammate spawn after this point in the orchestration is at risk of the same silent failure. Without an explicit follow-up `SendMessage`, the orchestrator wastes a spawn round and burns context diagnosing what looks like a stuck teammate.

**Where it lands.** `team-impl-v2.md` → `Story Implementation Cycle` → `1. Spawn the Implementer` (lines ~303–311). Same pattern in `2. Verification` (line ~317) for reviewer spawns.

**Proposed change.** Append to `1. Spawn the Implementer`:

> **Spawn-then-deliver.** Team-spawned teammates do not reliably receive the `prompt` argument from `Agent({team_name, ...})`. Immediately after the spawn call, follow with `SendMessage({to: <teammate-name>, message: <full handoff prompt>})`. The handoff is delivered via SendMessage, not via the spawn-time `prompt` field. If a freshly spawned teammate emits 3+ idle notifications in quick succession with no substantive message, assume the inbox is empty — re-deliver the handoff via SendMessage before considering any other failure mode.

Apply the same one-sentence reminder in `2. Verification` where the reviewer is spawned.

---

## Refinement 2 (A2) — Disambiguate idle notifications into two real failure modes

**Pilot evidence.** Two failure modes from the same run:

- *A.* First teammate spawn emitted 4 idle notifications in 11 seconds (caused by Refinement 1; fix: re-deliver via SendMessage).
- *B.* Across 15+ minutes of active work, a teammate emitted periodic idle notifications every 2–3 minutes while its background process produced hundreds of events. The teammate was actively working; the idles were turn-boundary noise.

**Why it matters.** Both modes share the same signal (idle notification) but require opposite responses (re-deliver vs. ignore). The current `Idle Notifications Are Unreliable Signals` operational pattern (lines ~503–508) treats idle as one ambiguous signal. Conflating the two modes is the original methodology bug.

**Where it lands.** `team-impl-v2.md` → `Operational Patterns` → `Idle Notifications Are Unreliable Signals` (lines ~503–508).

**Proposed change.** Replace the single-paragraph subsection with:

> ### Idle Notifications Are Unreliable Signals
>
> Idle notifications fire on conversation-turn boundaries, not on stops. A teammate actively running a 15-minute background task will emit several idle notifications while the work is still in flight. A teammate whose mailbox is empty will also emit idle notifications. The signal alone does not distinguish the two.
>
> Use the timing pattern to disambiguate:
>
> - **Rapid burst immediately post-spawn (3+ in <30 seconds).** Empty mailbox — the spawn-time `prompt` did not arrive. Re-deliver the handoff via `SendMessage` before assuming any other failure (see Refinement 1).
> - **Periodic idles during long-running work (one every few minutes, mid-task).** Conversation-turn boundaries — the teammate is still working. Ignore. Wait for the teammate's explicit completion message.
> - **Sustained silence past the expected duration for the task.** Send one brief nudge: "Did you complete the work? Report your results." Don't conclude failure from silence alone.
>
> The reliable signal is the teammate's explicit message reporting results. The idle notification pattern is supporting context, not a primary signal.

---

## Refinement 3 (A3) — Remove the standalone-subagent trap

**Pilot evidence.** Caught twice on the project by user interruption. The phrase *"fire a `senior-engineer` subagent"* was read as a different spawn mechanism (standalone `Agent` without `team_name`), producing teammates with no mailbox, no task-list visibility, and a stricter permission posture.

**Why it matters.** The team contract is the load-bearing structure of the orchestration. Every dispatch — implementer, reviewer, fixer, one-shot — must spawn on the active team. The current text actively encourages the opposite for quick fixes, and reinforces it in section `7. Create Team`.

**Where it lands.** Three locations in `team-impl-v2.md`:

1. `The Orchestrator's Role` (lines ~41–42)
2. `Story Implementation Cycle` → `7. Create Team` (line ~252)
3. `Story Implementation Cycle` → `3. Orchestrator Final Check` (line ~330)

**Proposed change.**

In `The Orchestrator's Role`, replace:

> - Quick fixes (typos, one-file adjustments) → fire a `senior-engineer` subagent
> - More extensive work (multi-file changes, architectural adjustments) → spawn a new general-purpose teammate

with:

> - **All routed work — fixes large or small — spawns on the active team.** Use `Agent({team_name, name, subagent_type})` for every dispatch. Quick fixes (typos, one-file adjustments) and extensive work (multi-file changes, architectural adjustments) differ only in scope and teammate lifetime, not in spawn mechanism. Even a 3-line edit goes to a short-lived team-resident teammate, not a standalone subagent.
> - The `senior-engineer` phrasing in this skill refers to a worker's *type*, not to a distinct spawn mechanism. There is no "subagent path" parallel to the team path.

In `7. Create Team`, replace the existing senior-engineer line with:

> All teammates — implementers, reviewers, one-shot fixers — are spawned on the active team via `Agent({team_name, name, subagent_type})` with bypassPermissions. Standalone `Agent` calls (without `team_name`) are never used during orchestration. They produce workers with no mailbox, no task-list visibility, and a stricter default permission posture, and they break every observability and routing guarantee this skill depends on.

In `3. Orchestrator Final Check` step 2, replace *"Route any remaining fixes to a senior-engineer subagent (quick) or fresh teammate (extensive)"* with *"Route any remaining fixes to a fresh team-resident teammate (short-lived for quick fixes, longer-lived for extensive work)."*

---

## Refinement 4 (A4) — Teammate lifecycle: silence on failure, SendMessage to exited teammate

**Pilot evidence.** Two adjacent failure modes from one incident:

- *A.* Driver exited with an error verdict; auto-report existed on disk; teammate stayed silent. Team-lead discovered the failure only by filesystem polling after a second idle burst.
- *B.* After teammate exited, team config `members[]` shrank. Subsequent `SendMessage` to the exited teammate's name silently enqueued into a nonexistent inbox.

**Why it matters.** Both modes share a common cause: the orchestration assumes an active relationship with a named teammate when there isn't one. (A) is a template-text problem — "relay on success" is read literally. (B) is an invariant the orchestrator must check before SendMessage.

**Where it lands.** Two locations in `team-impl-v2.md`:

1. Implementer template `Step 7` (lines ~189–198); equivalently in reviewer template `Step 7` (lines ~236–246).
2. `Operational Patterns` (new subsection after `CLI Reviewed Wrong Target`).

**Proposed change.**

Replace the leading instruction in implementer template `Step 7` (and equivalently in reviewer template `Step 7`) with:

> Step 7 — Report to orchestrator (SEND THIS MESSAGE on **every** driver exit — success, failure, partial, or unknown verdict):
>
> The terminating action of the implementer is the report message, regardless of the driver's verdict. Never stay silent on failure. If the driver auto-generated a report file on disk, summarize and reference it; do not assume team-lead will discover the failure independently. After sending the report, remain idle in case team-lead has follow-up — do not exit the teammate immediately.

(The bullet list of report fields stays as-is.)

Add a new `Operational Patterns` subsection after `CLI Reviewed Wrong Target`:

> ### Verify the Teammate Exists Before SendMessage
>
> When a teammate's driver exits, the team config's `members[]` shrinks. The teammate name remains a valid `SendMessage` target *syntactically*, but the message enqueues into a nonexistent inbox and is silently dropped. There is no delivery-failure signal.
>
> Before any `SendMessage` to a named teammate (especially mid-story for a fix round), check that the target appears in the current team config's `members[]`. If it does not, the teammate has exited — spawn a fresh teammate for the work rather than messaging the dead name.
>
> The orchestrator's instinct to reuse a recently exited teammate ("their context is already loaded") is a trap. The replacement teammate reads the necessary artifacts cold, which is the same pattern fresh-agents-per-story enforces.

---

## Refinement 5 (A5) — Pre-acceptance fabrication scan with project-specific pattern inventory

**Pilot evidence.** A CLI subagent without network access fabricated `link:` file-path dependencies pointing at arbitrary local directories outside the workspace, producing a locally-green but unportable repo. Caught by orchestrator file-state spot-check, not by gates or review.

**Why it matters.** When a CLI subagent hits a silent environmental limit (network disabled, tool missing, permission denied), it can autonomously fabricate a workaround that passes the structural gate. Dual review passes too, because reviewers check faithfulness-to-spec, not implementer-environment-truthfulness. The structural gate in the orchestrator is the only backstop.

**Where it lands.** `team-impl-v2.md` → `Story Implementation Cycle` → `3. Orchestrator Final Check` (lines ~327–333). Insert a new step between current step 3 and current step 4 (renumber accordingly).

**Proposed change.**

> **4. Scan for CLI-fabrication patterns.** When a CLI subagent hits an environmental limit silently (network disabled, tool missing, permission denied), it may fabricate a local workaround that looks structurally green. Maintain a project-specific list of "impossible outputs" — outputs that should never appear in committed source if the subagent operated honestly — and grep for them as a deterministic pre-acceptance check. Materialize the list in `team-impl-log.md` alongside the boundary inventory so it accumulates across stories and survives context stripping. Examples to seed the list:
>
> - `link:` paths in package manifests pointing outside the workspace
> - Absolute host paths in committed configs
> - Unresolved environment-variable literals (`${...}` patterns) in committed files where they should have been substituted
> - Hardcoded localhost URLs in code that should use config
>
> This gate catches "the subagent worked around a problem instead of reporting it" — neither the structural acceptance gate nor dual review reaches that class of defect.

---

## Refinement 6 (A6) — Delivery-assertion marker for every CLI prompt

**Pilot evidence.** A path-resolution divergence caused the CLI to receive a 0-byte stdin. The CLI produced a 558 KB, 248-event transcript with command-execution items, a turn.completed event, and a 31K-token agent message claiming 26 files created. `git status` showed nothing. ~25 minutes wasted before the orchestrator realized the CLI had never seen the prompt.

**Why it matters.** Highest-confidence catch in the findings list because the failure is silent and the cost per occurrence is high. Every CLI dispatch is exposed to it. The fix is four lines of orchestration that deterministically catch the failure on round one.

**Where it lands.** `team-impl-v2.md` → both teammate templates: implementer template `Step 3` (lines ~174–182) and reviewer template `Step 3` block A (lines ~216–227); plus a brief reference in `External Model Failure Protocol` (lines ~282–296).

**Proposed change.**

In the implementer template's `Step 3`, append:

> **Delivery-assertion marker (mandatory).** Every CLI prompt must open with: *"First thing: echo `<UNIQUE_MARKER>` before anything else."* The marker uses the standardized prefix `LSV2-MARKER-` plus a `<timestamp>-<random>` suffix (e.g., `LSV2-MARKER-20260419-a3f7c1`). After the CLI exits, the teammate verifies the marker appears at least once in the recorded session output **using the selected CLI skill's standard output-extraction mechanism** — the surfaces exposed by `codex-subagent`, `copilot-subagent`, or whichever CLI skill is loaded for this run. Do not hard-code a shell command (`grep`, `findstr`, etc.) for this check: shell commands are brittle across platforms (Windows hosts often lack `grep`) and they bypass the structured extraction the CLI subagent skills already standardize. If the marker is absent from the recorded output, the prompt did not reach the CLI — stop, do not trust any reported results, and escalate the delivery failure to team-lead before retrying. This is the only gate that catches silent stdin/path-delivery failures and the fabrications they enable.

In the reviewer template's `Step 3` block A, append the same paragraph (substituting "the reviewer's CLI run" wording where needed).

In `External Model Failure Protocol` (lines ~282–296), after step 3:

> If a teammate reports CLI failure but their session output shows a successful-looking run, check for the delivery-assertion marker via the CLI skill's extraction mechanism before retrying. A successful-looking session with no marker is empty-prompt fabrication, not a CLI failure — diagnose the delivery path (stdin, file argument, working directory) before assuming the CLI is the problem.

---

# Group B — `ls-tech-design-v2` authoring hardening

Refinement 7 split into two related sub-refinements that close two distinct defect classes.

## Refinement 7a (B1) — Runtime-validatable toolchain config snippets (config-shape defects)

**Pilot evidence.** Five concrete config-shape incidents:

*From the findings doc (Story 7):*

- An empty "reserved slot" the build tool rejected as invalid
- An adjacent section missing a required `rollupOptions.input`
- A root `package.json` missing a required `main` field

*From `decisions-log.md` (Story 8):*

- `.npmrc` missing `node-linker=hoisted` — symlinked pnpm layout works for dev and `pnpm start`, breaks at first packaging
- `electron-vite` renderer `outDir` with `..`-relative path silently fails on Windows under Vite 8 — stdout claims success, disk shows nothing

All five invisible to dual review (which checks faithfulness-to-spec, not tool-shape correctness) and to the automated gate (`pnpm verify`, format, lint, typecheck, unit tests don't exercise bundler-construction or packaging codepaths).

**Why it matters.** This is a class of defect the spec itself carries. The implementer copies the snippet faithfully; the snippet is wrong against the tool's runtime requirements. The fix lives at the authoring layer.

**Where it lands.** New subsection in `tech-design-v2.md` immediately before `Validation Before Handoff` (insertion point: after `Test Count Reconciliation`, line ~393).

**Proposed change.**

> ## Toolchain Config Snippet Validation
>
> Tech-design documents prescribe configuration snippets — bundler configs, dev-server configs, test-runner configs, framework configs. Implementers copy them verbatim. Dual review checks faithfulness-to-spec, not tool-shape correctness. The result is a class of defect the spec itself carries: empty reserved slots the tool rejects as invalid, missing required fields the tool needs at runtime, format inconsistencies the tool's parser refuses to load. None of these surface until the prescribed end-to-end command actually runs.
>
> Before handoff, validate every prescribed config snippet against the tool's real shape requirements:
>
> 1. **Constraint annotation (lighter weight, always required).** For every snippet, ask: *"Would this validate as-is if the tool ran against it right now?"* Annotate the snippet with the tool's minimum-required shape at that level (e.g., "every Rollup input must have at least one entry," "every Vite plugin entry must have a `name` field," "the root `package.json` must define `main` for the build output to be locatable"). The annotation forces the author to read the snippet under the tool's lens, which catches "empty slot" and "reserved for future" defects at spec time.
> 2. **Synthetic-repo validation (heavier weight, when feasible).** When the prescribed toolchain is well-known and self-validating, run the tool against a synthetic repo built from the snippet and confirm it starts. Not always practical, but cheap for framework-level configs like bundlers and dev servers.

Add to the `Validation Before Handoff` checklist (after the existing dependency-research item):

> - [ ] Every prescribed toolchain config snippet annotated with its minimum-required shape; runtime-validation step performed where feasible

## Refinement 7b (B2) — Observed-run gate is load-bearing for runtime-path defects (distinct class)

**Pilot evidence.** From `decisions-log.md` (Story 8): packaged ESM Electron 41 main with TLA chain in `app.whenReady()` hung indefinitely. Dev mode unaffected because dev didn't use the same TLA sequence. The config snippet itself was correct; the runtime *path* the snippet triggered behaved differently from the dev-mode path.

**Why it matters.** This is a different defect class from 7a. Even a perfect config-snippet review pass cannot catch defects whose root cause is a runtime path the spec exercised differently in dev than in the deployed surface. The observed-run gate — a human running the prescribed end-to-end command at least once before story acceptance, against the actual runtime artifact — is the only surface that catches this class. Distinguishing it from 7a's config-shape defects matters because the two have different fixes: 7a tightens the spec; 7b enforces a gate.

**Where it lands.** Two locations across both v2 sources — declaring the gate is not enough; it has to be enforced where acceptance actually happens.

1. `tech-design-v2.md` → append to the new `Toolchain Config Snippet Validation` subsection (immediately after Refinement 7a's content) — *spec-authoring layer: the tech design declares which observed-run commands are part of the gate set*
2. `team-impl-v2.md` → `Verification Gate Discovery` (lines ~89–103) and `3. Orchestrator Final Check` (line ~327) — *acceptance-path enforcement layer: the orchestrator locks the observed-run gate into the discovered gate set during setup, and runs it as part of story acceptance*

Without the team-impl-v2.md landings, the tech-design declaration is advisory: the orchestrator could lock in `pnpm verify` (or whatever the project's structural gate is) as the only acceptance command and never run the prescribed observed-run check. Story and epic acceptance are owned by `team-impl-v2.md`'s `Verification Gate Discovery` and `3. Orchestrator Final Check` flows — the gate has to be enforced there, not just declared in the tech design.

**Proposed change — `tech-design-v2.md`.**

> **Load-bearing corollary: the observed-run gate is not advisory.** Some defects are not config-shape problems and cannot be caught by snippet annotation. Runtime paths that differ between dev and packaged surfaces — top-level-await chains that resolve in one and hang in the other, code paths only reached by the production startup sequence, behaviors that depend on the actual deployment artifact — are invisible to every static review. The observed-run gate (a human running the prescribed end-to-end command at least once before story acceptance, against the actual runtime artifact) is the only surface that reaches this class. It belongs in the acceptance path, not as optional polish.
>
> When the tech design prescribes an observed-run gate, name the exact command(s), the runtime artifact they execute against (e.g., the packaged installer, not the dev server), and whether the gate applies to story acceptance, epic acceptance, or both. The orchestration skill (`team-impl-v2.md`) then locks those commands into the gate set during `Verification Gate Discovery` and runs them as part of story acceptance.
>
> Refinement 7a addresses *config-shape* defects (the snippet's text is wrong against the tool's parser). Refinement 7b's gate addresses *runtime-path* defects (the snippet is fine, but the runtime path it triggers behaves differently from the path dev validation exercised). Both fixes are required because they close different defect classes; better spec review does not replace running the actual runtime path at least once.

**Proposed change — `team-impl-v2.md`, `Verification Gate Discovery` (lines ~89–103).**

After step 2 of the existing text (which already discovers the story acceptance gate and epic acceptance gate from project policy docs), insert a new step:

> 2.5. **If the tech design specifies observed-run gates** (per Refinement 7b in `tech-design-v2.md`), add them to the story acceptance gate, the epic acceptance gate, or both as the tech design prescribes. Lock the exact commands and the runtime artifact they execute against (e.g., the packaged installer, not the dev server) into `team-impl-log.md` alongside the project-policy gates. Observed-run gates exist precisely to catch defects the structural gate does not exercise — packaged-only runtime paths, deployment-artifact behaviors, top-level-await chains that resolve in dev but hang in production. They are not optional polish; they are the only surface that catches their defect class. If a tech design prescribes an observed-run gate and the orchestrator omits it from the locked gate set, the gate discovery is incomplete.

**Proposed change — `team-impl-v2.md`, `3. Orchestrator Final Check` (line ~327).**

Update step 1 of the existing text (which currently says "Run the discovered story acceptance gate yourself"):

> 1. **Run the discovered story acceptance gate yourself** — execute the exact commands you locked in Verification Gate Discovery and confirm they pass. This includes any observed-run gate the tech design prescribed (per Refinement 7b); structural-gate passes do not substitute for an observed run of the prescribed runtime path. Don't trust reports alone.

---

# Group C — Joint UI-companion hardening across `ls-tech-design-v2` and `ls-team-impl-v2`

Five sub-refinements addressing the verification surface as the load-bearing UI-companion seam. The five together establish enforcement at multiple layers: spec-authoring (8, 8b, 8c), implementer dispatch (8), reviewer dispatch (8), and human-gate framing (8d).

## Refinement 8 (C2 + C3) — UI-spec verification surface must specify capture mode (three-layer enforcement)

**Pilot evidence.** 17 Playwright baselines per the UI-spec state matrix. All 17 passed structural review. The footer component existed, rendered, composed in the right place — but sat below the default 1280×800 fold. The default-viewport baselines clipped the footer. Caught by the human visual-review gate on a lucky pass, per `docs/findings/skill-refinements.md`. The post-fix screenshots in `__screenshots__/` corroborate the recovered shape (full-page captures).

**Why it matters.** The verification surface was named ("Playwright screenshot capture") but its non-default capture mode was not declared. Playwright's default is viewport-only; the spec needed full-page. Structural review still passed (named components existed, named states reachable, screenshots produced) — but the screenshots were silently truncated.

**Where it lands.** Three locations:

1. `tech-design-v2.md` → `Verification Surface Requirement` subsection in the UI Companion section (lines ~526–537) — spec-authoring layer
2. `team-impl-v2.md` → implementer template (lines ~159–164 and ~177–181) — implementer dispatch layer
3. `team-impl-v2.md` → reviewer template (lines ~218–227) and `UI Spec Verification Ceiling` operational pattern (lines ~552–558) — reviewer dispatch layer

**Proposed change.**

### Layer 1 — `tech-design-v2.md`, `Verification Surface Requirement`

Append two paragraphs after the existing "screenshots do double duty" line:

> **Capture-mode default: full page, deterministic.** When the verification surface is screenshot capture, the default for every named state is full-page capture (`fullPage: true` or the tool-equivalent option) with deterministic options enabled (animations disabled, fonts loaded, network idle reached). Viewport-only captures are an explicit exception — they are named in the state matrix as `above-the-fold`, `header-only`, or another descriptive label, with the capture mode annotated alongside the state name. Default-viewport capture on a full-page state silently truncates content below the configured fold; the structural compliance check still passes, but the visual-review surface is incomplete.
>
> **Width-vs-rectangle distinction.** When a state names a responsive width (e.g., `<state> at 960×600`), the dimension specifies the minimum viewport target, not the capture rectangle. Baselines capture full-page at the specified width, not clipped to the specified height.

### Layer 2 — `team-impl-v2.md`, implementer template

Update `Step 2` reading-journey UI-spec instructions (line ~159–164):

> 3. [UI spec, if present] — Read. Reflect: which screens and states does this story touch? What component contracts (referencing tech-design identifiers) and state presentations must the implementation honor? What is the project's chosen verification surface (typically Playwright screenshots)? **What capture mode does the spec name for each state — full-page (default) or an explicit viewport-only exception?** What screenshots does this story need to produce, and at what capture mode?

Update `Step 3` UI-spec block (lines ~178–181):

> If a UI spec is present, instruct the CLI to honor the component contracts and state presentations from the spec, reference tech-design identifiers (don't redefine them), and produce the verification-surface artifacts the spec specifies. **For Playwright (or any screenshot tool), the CLI must capture full-page (`fullPage: true`) with deterministic options (animations disabled, fonts loaded, network idle reached) for every state unless the spec explicitly marks that state as viewport-only. A default-viewport capture on a full-page state is a defect, not a successful capture.**

Update `Step 7` UI-spec line (line ~196):

> - If a UI spec was in scope: which named states have screenshots produced (with capture mode noted per state), which do not yet, and why

### Layer 3 — `team-impl-v2.md`, reviewer template + operational pattern

Add to reviewer template `Step 3` UI-spec checks (lines ~218–227):

>        - **Capture mode is full-page (`fullPage: true` or tool-equivalent) for every state unless the spec explicitly marks it as viewport-only. Deterministic options are enabled. A viewport-only capture on a full-page baseline is a Major finding.**

Update reviewer template `Step 7` UI-spec line (line ~243):

> - If a UI spec was in scope: per-screen state coverage status, screenshot artifact list (paths or counts, with capture mode noted per state), one-way ownership contract status

Append to `UI Spec Verification Ceiling` operational pattern (lines ~552–558):

> **Capture-mode discipline.** When the verification surface is screenshot capture, every state captures full-page with deterministic options unless the spec explicitly marks the state as viewport-only. Default-viewport capture on a full-page state silently truncates content; structural review still passes, the human visual review is the only backstop, and the backstop is unreliable when the reviewer doesn't know what they're missing. The capture-mode requirement is enforced at three layers: the spec names the default (`tech-design-v2.md` Verification Surface Requirement), the implementer template requires `fullPage: true`, the reviewer flags violations as Major.

---

## Refinement 8b (C4) — Narrow general principle: declare tool defaults that silently narrow the human-review surface

**Pilot evidence.** Refinement 8 is an instance of one specific failure pattern: when a verification tool default would silently narrow what the human reviewer can see, structural compliance still passes but the human-visible surface is incomplete. Playwright's full-page-vs-viewport default was the one that bit this pilot. **No other instances have been observed yet** in this repo.

**Why it matters.** Stating the narrow rule once captures the lesson without creating cargo-cult ceremony. Anchoring it to *what humans can review* — not to "every tool option" — keeps the principle scoped to the actual failure mode.

**Important: this refinement deliberately stays narrow.** Earlier drafts proposed extending the rule into a cross-tool taxonomy covering accessibility severity thresholds, snapshot diff sensitivity, font loading, network idle, and other defaults. The current evidence does not yet justify that broader formulation. If a second pilot surfaces the same failure shape on a different tool class, that broader principle can be promoted then; until then, the narrow rule alone is what is earned.

**Where it lands.** `tech-design-v2.md` → `Verification Surface Requirement` subsection (same location as Refinement 8's main edit).

**Proposed change.** Add a paragraph after the Refinement 8 additions:

> **Narrow general principle: declare tool defaults that silently narrow the human-review surface.** When the spec names a verification tool, every default whose effect is to narrow what the human reviewer can see must be declared in the spec as a non-default. The principle is anchored to the *human-review surface*, not to "every tool option" — the failure mode is silent narrowing of what reaches the reviewer, not configuration completeness. Screenshot capture mode (full-page vs. viewport-only) is the concrete instance that proved the rule. If later pilots surface the same shape on a different tool class, this principle extends to that class at that point; until then, screenshot capture mode is the only application required by current evidence.

---

## Refinement 8c (C1) — Verification Surface gets a stable home in the artifact (admitted contract adjustment)

**Pilot evidence.** The produced `ui-spec.md` followed the v2 8-section template verbatim for sections 1–8, then added a separate **Verification Surface** section after Section 8 codifying the chosen tool, the state-by-state matrix, the state-driving mechanism, and any spike requirements.

**Why it matters.** The v2 skill body already requires a verification surface to be specified (Phase 5 checklist + Verification Surface Requirement subsection). The artifact template (the 8-section list in Output Artifact) doesn't tell the author *where in the artifact* that specification lives. The pilot author resolved the ambiguity by appending a trailing section. The next author will resolve it some other way without explicit guidance.

**This is a v2 contract adjustment, explicitly admitted.** Earlier drafts of this refinement tried to call the trailing section "not really a shape change" or "not a 9th section." That framing is dishonest — adding a required trailing section after the 8 named sections **is** a small contract adjustment. This proposal adopts it deliberately and provisionally for the next pilot, and does not pretend the artifact contract is unchanged.

**Provisional, not permanent.** The trailing-section shape is a v2 working convention for the next pilot, not a public permanent doctrine claim. The first pilot resolved the placement ambiguity this way; if the second pilot resolves it differently and that resolution proves clearer, fold the better shape back into this template. Until then, the trailing-section convention removes the placement ambiguity downstream consumers (the implementer template, the reviewer template, the human visual-review gate) currently inherit.

**Where it lands.** `tech-design-v2.md` → `Output Artifact: ui-spec.md` subsection (lines ~539–552), specifically the paragraph after the 8-section list.

**Proposed change.** Replace the existing "Sections beyond these..." paragraph with:

> Sections beyond these (Component Reuse Matrix, Token and Theme Impact as separate sections, Layout and Responsive Rules as separate sections, Visual Direction, Issues Found, etc.) are deliberately deferred for v2. Fold their essentials into Component Specifications when the feature calls for it; split them out into named sections only when their absence causes observable downstream errors. The 8-section shape is intentional — earn complexity, don't front-load it.
>
> **Required trailing section: Verification Surface (v2 contract adjustment, provisional).** When the companion is produced, the artifact carries an explicit **Verification Surface** section after Section 8 that codifies what the body's Verification Surface Requirement settled. Adding this required trailing section is a small adjustment to the v2 artifact contract, adopted for the next pilot to remove the placement ambiguity the first pilot exposed.
>
> Minimum required contents:
>
> - The chosen verification tool/surface
> - The state-by-state matrix where applicable (state × variant if relevant)
> - Any non-default tool options being declared (per Refinement 8b's narrowing principle)
> - The state-driving mechanism the implementation must support (e.g., DEV-only query-string flags or pre-seeded fixtures)
> - Any setup/spike prerequisites before the surface can run
>
> The exact section structure beyond these required contents is left to the author. The shape this section took in the first pilot was workable but is not yet promoted into permanent doctrine — revisit after a second pilot to see whether the same shape emerges or a clearer one does.

---

## Refinement 8d (C5) — Sharpen the human visual-review gate's role

**Pilot evidence.** The current v2 text is honest that agents verify *structural* UI compliance only and that visual quality remains a human gate. After the Refinement 8 / 8b / 8c changes land, the human gate's role can be framed more precisely.

**Why it matters.** Without 8d, the human gate could be misread as the only backstop for everything the methodology might have missed (including silent truncation, undeclared tool defaults, missing deterministic options). With 8d, the human gate is correctly framed as the *final arbiter on a methodology-validated surface*. The methodology eliminates avoidable upstream errors before the surface reaches the human; the human still owns the final judgment on visual polish, hierarchy, and rhythm.

**Where it lands.** `team-impl-v2.md` → `UI Spec Verification Ceiling` operational pattern (lines ~552–558). Sits adjacent to Refinement 8's Layer 3 capture-mode-discipline paragraph.

**Proposed change.** After Refinement 8's "Capture-mode discipline" paragraph, append:

> **The human gate's role, sharpened.** With Refinements 8 / 8b / 8c in place, the human visual-review gate is the *final arbiter of visual quality on a methodology-validated surface*. It is not the only backstop for methodology gaps. The methodology's job is to eliminate avoidable upstream errors before the surface reaches the human — silent truncation, undeclared tool defaults, missing deterministic options — so that what the human reviews is the *intended* surface, not whatever the tool happened to capture by default. The human still owns the judgment on polish, hierarchy, rhythm, and hover-state feel; the methodology owns the completeness of the surface that judgment is exercised against.

---

# Resolved defaults

Five operational choices that earlier drafts left as open questions are resolved here so the implementation pass does not have to make methodology decisions on the fly.

1. **One-shot fixers stay on the active team but do NOT require a new materialized template.** The implementer and reviewer templates remain the only stable templates in `team-impl-log.md`. Ad-hoc fixer dispatches are constructed inline at orchestration time. This keeps the template inventory small and stable while preserving the team-only spawn discipline from Refinement 3.

2. **The `members[]` existence check is an Operational Pattern, not a Hard Invariant.** The Control Contract holds invariants tied to verification *evidence* (no acceptance without external-model verification, no unresolved findings without disposition, no silent degradation). The membership check is routing discipline, which is the Operational Patterns layer's territory. This altitude is correct.

3. **The fabrication-pattern list lives in `team-impl-log.md`** alongside the boundary inventory. It accumulates across stories and survives context stripping, paralleling the boundary-inventory pattern Refinement 5 references.

4. **The CLI delivery marker uses the standardized prefix `LSV2-MARKER-<timestamp>-<random>`.** Standardized prefix makes the marker easier to grep across logs; per-dispatch random suffix prevents accidental cross-dispatch matches.

5. **Refinement 8c is provisional, not permanent.** Adopted as a v2 working convention for the next pilot; revisited at pilot 2's retrospective.

---

# Items explicitly deferred or rejected

Six points came up during the proposal/review cycle and are intentionally **not** included as accepted changes here. Recording them so the next pass does not re-litigate the same questions.

1. **Do not claim the `ui-spec.md` artifact shape is unchanged.** Earlier drafts tried to frame Refinement 8c as "not really a shape change." That framing is dishonest. Adding a required trailing section is a small v2 contract adjustment; this proposal admits it explicitly.

2. **Do not use the packaged-only `app.whenReady()` hang as evidence for config-snippet validation.** That incident supports Refinement 7b (observed-run gate), not Refinement 7a (snippet annotation). Lumping it under 7a weakens the causality of the refinement.

3. **Do not generalize Refinement 8b into a broad cross-tool verification taxonomy.** The narrower "human review surface" rule is accepted. The broader accessibility/snapshot taxonomy is deferred until a second pilot surfaces the same failure shape on a different tool class.

4. **Do not say the screenshot directory alone proves the original truncation defect.** The direct evidence comes from `docs/findings/skill-refinements.md`. The screenshot directory corroborates the *fix shape*, not the original failure. Signal 2 in this proposal is phrased to reflect that distinction.

5. **Do not require a mirrored "Validation Before Handoff Checklist" inside the trailing Verification Surface section.** Earlier drafts proposed this. The combined feedback does not yet support making it required in v2. The Phase 5 checklist in the skill body is sufficient until a second pilot demonstrates otherwise.

6. **Do not restructure the existing three Control Contract invariants.** Refinements 6 and 7b add two short addenda extending invariants 1 and 3; they do not rewrite the existing invariants or change their numbering. Major Control Contract reorganization is out of scope for v2 and would require its own proposal cycle.

---

# Cross-cutting decisions

A few choices apply across multiple refinements.

### One bundled change

All refinements ship as one change to `src/phases/team-impl-v2.md` and `src/phases/tech-design-v2.md`. The actual edits land together once this proposal is approved. The implementation pass does not start until then.

### v2-only, no v1 propagation now

Refinements 1–7b describe failure modes that exist in v1 too. v1 is not edited as part of this change. v1 propagation happens at the v2 experiment's resolution point along with the rest of the v2 deltas.

### Multi-layer defense for UI-companion changes

Refinement 8 lands at three layers (spec author → implementer → reviewer). Refinements 8b, 8c, 8d add complementary layers (narrow generalization, artifact placement, human-gate framing). Naming a discipline at only one layer leaves it advisory; naming it at multiple makes it enforceable. This pattern should be preserved for any future UI-companion discipline that needs to survive context decay across spawn boundaries.

### Operational Patterns vs. Control Contract

Refinements 1, 2, 4-membership, and 8d land in `Operational Patterns` because they are routing-discipline edits, not evidence-claim edits.

**Refinements 6 and 7b are evidence-claim edits** and are placed at the right altitude accordingly. The earlier "none of these eight refinements change acceptance evidence" framing was incorrect; this section corrects it.

- **Refinement 6's marker-presence rule** sharpens existing Control Contract invariants 1 and 3 (no acceptance without verification evidence; no silent degradation). The proposed change text lands the rule operationally in both teammate templates (so the check executes per CLI dispatch). It also adds a one-paragraph addendum to the Control Contract noting that *verification evidence requires positive proof of CLI invocation; an unmarked recorded session is fabrication, not evidence*. The addendum extends invariant 3 — it does not rewrite the existing invariants.
- **Refinement 7b's observed-run requirement** changes what counts as a complete acceptance gate. It lands in `Verification Gate Discovery` (so the gate set is locked correctly during setup) and in `3. Orchestrator Final Check` (so the orchestrator actually runs the prescribed observed-run command before story acceptance). It also adds a one-paragraph Control Contract addendum noting that *when the tech design prescribes an observed-run gate, structural-gate passes alone do not satisfy story or epic acceptance*. The addendum extends invariant 1 — it does not rewrite the existing invariants.

Both addenda are intentionally short. The hardening pass extends the Control Contract; it does not restructure it. Major reorganization of the existing three invariants is out of scope for v2 and is listed under Items Explicitly Deferred or Rejected.

**Proposed addendum text (single block, append to the end of `Control Contract (Hard Invariants)` in `team-impl-v2.md`, around line ~258):**

> **Addendum (v2 hardening pass):**
>
> - *Marker-presence requirement (extends invariant 3).* Verification evidence from a CLI dispatch requires the delivery-assertion marker (Refinement 6) to be present in the recorded session output, verified via the selected CLI skill's extraction mechanism. An unmarked session is empty-prompt fabrication, not verification, regardless of how successful the transcript looks.
> - *Observed-run gate requirement (extends invariant 1).* When the tech design prescribes an observed-run gate (Refinement 7b), the gate's commands are part of story or epic acceptance per the tech design's scope assignment. Structural-gate passes alone do not satisfy acceptance for stories or epics that prescribe observed-run gates. The orchestrator runs the prescribed observed-run commands during `3. Orchestrator Final Check` after locking them into the gate set during `Verification Gate Discovery`.

### Refinement 7's two-part split is intentional

7a (config-shape defects) and 7b (runtime-path defects) close different classes the original findings doc lumped together. Both must land; neither subsumes the other. The "Load-bearing corollary" framing in 7b makes the relationship explicit.

---

# Implementation touchpoints

Source anchors for the drafting pass.

### `src/phases/team-impl-v2.md`

- Line ~25 — `The Orchestrator's Role` (Refinement 3)
- Line ~89 — `Verification Gate Discovery` (Refinement 7b — observed-run gate locked into gate set)
- Line ~142 — implementer template (Refinements 4, 6, 8)
- Line ~200 — reviewer template (Refinements 4, 6, 8)
- Line ~252 — `7. Create Team` (Refinement 3)
- Line ~258 — `Control Contract (Hard Invariants)` (Refinements 6, 7b — addenda extending invariants 1 and 3)
- Line ~282 — `External Model Failure Protocol` (Refinement 6)
- Line ~303 — `1. Spawn the Implementer` (Refinement 1)
- Line ~317 — reviewer-spawn guidance in `2. Verification` (Refinement 1)
- Line ~327 — `3. Orchestrator Final Check` (Refinements 3, 5, 7b — observed-run gate enforced at acceptance)
- Line ~503 — `Idle Notifications Are Unreliable Signals` (Refinement 2)
- Line ~552 — `UI Spec Verification Ceiling (v2-specific)` (Refinements 8, 8d)

### `src/phases/tech-design-v2.md`

- Line ~393 — insertion point for new `Toolchain Config Snippet Validation` subsection (Refinements 7a, 7b)
- Line ~396 — `Validation Before Handoff` checklist (Refinement 7a)
- Line ~526 — `Verification Surface Requirement` (Refinements 8, 8b)
- Line ~539 — `Output Artifact: ui-spec.md` (Refinement 8c)

---

# Recommended drafting order

1. Draft the orchestration changes in `team-impl-v2.md` first (Refinements 1–6). These have the cleanest insertion points and the lowest risk of cross-cutting interactions.
2. Draft the config-validation and observed-run clarifications in `tech-design-v2.md` (Refinements 7a, 7b). These add a new subsection and one checklist item — minimal interaction with existing content.
3. Draft the UI-companion changes (Refinements 8, 8b, 8c, 8d) — both files involved, three layers in `team-impl-v2.md`, two locations in `tech-design-v2.md`. Most cross-cutting; do last so the prior changes are stable.
4. Run `bun run verify` (build + validate + tests).
5. Spot-check `dist/skills/ls-tech-design-v2/SKILL.md` and `dist/skills/ls-team-impl-v2/SKILL.md` for composition coherence.
6. Reinstall the v2 pair into `~/.claude/skills/`:
   ```bash
   cp -r dist/skills/ls-tech-design-v2 ~/.claude/skills/ls-tech-design-v2
   cp -r dist/skills/ls-team-impl-v2 ~/.claude/skills/ls-team-impl-v2
   ```

---

# Pilot 2 readiness

After the implementation pass, the next pilot exercises the refinements. Capture during the run:

- Did each refinement fire as intended? (Specifically: did A1's spawn-then-deliver remove the empty-mailbox failure mode? Did A6's marker grep catch any silent delivery failures? Did C1's trailing-section shape from the first pilot hold, or did the next author re-derive it differently?)
- Did any new failure mode appear that none of these eight refinements address?
- For Promote/Merge eligibility per the v2 plan's decision matrix, the second pilot should be a brownfield project to exercise the Inherit and Extend strategies the first (greenfield) pilot did not test.

The decision matrix in `docs/research/ls-tech-design-v2-plan.md` is unchanged. The combined evidence from pilots 1 and 2 feeds into the Promote / Merge / Retire decision at that point.

---

# Recommendation

This is the single proposal of record for the next v2 hardening pass. No further proposal-level discussion is required before drafting begins. The defaults in **Resolved defaults** and the rejections in **Items explicitly deferred or rejected** are sufficient to start the implementation pass when the owner is ready.

---

# Review comments — resolution log

The three review comments on the previous revision have been addressed. Recording the resolutions here so the audit trail survives.

1. **Major — Refinement 7b does not yet reach the skill that actually owns acceptance.** *Resolved.* Refinement 7b's "Where it lands" now names two locations: `tech-design-v2.md` for spec-authoring (declare the observed-run gate) and `team-impl-v2.md` for acceptance-path enforcement (lock the gate into `Verification Gate Discovery`, then run it during `3. Orchestrator Final Check`). New proposed-change text was drafted for both team-impl-v2.md insertion points. Implementation Touchpoints updated to list the new line numbers (~89 and the extension at ~327). The corresponding Control Contract addendum (extending invariant 1) is also added at line ~258.

2. **Medium — Refinement 6 hard-codes a Unix-style shell check into a cross-CLI orchestration skill.** *Resolved.* The proposed-change text for Refinement 6 no longer references `grep -c` or any specific shell command. The durable rule is now: *verify the marker appears at least once in the recorded session output using the selected CLI skill's standard output-extraction mechanism* (the surfaces exposed by `codex-subagent`, `copilot-subagent`, or whichever CLI skill is loaded). The brittleness of shell commands across platforms (Windows hosts often lack `grep`) and the bypass of CLI-skill structured extraction are both called out explicitly. The External Model Failure Protocol reference was updated to use the same framing.

3. **Medium — The proposal understates its own acceptance-surface changes.** *Resolved.* The Goal section now admits **three** contract-level changes (artifact shape from 8c, acceptance-evidence trustworthiness from 6, acceptance-path observed-run requirement from 7b) instead of "exactly one small way." The Cross-cutting decisions section has a rewritten "Operational Patterns vs. Control Contract" subsection that explicitly classifies Refinements 6 and 7b as evidence-claim edits and proposes two short Control Contract addenda — one extending invariant 3 (marker presence required for trustworthy CLI evidence), one extending invariant 1 (observed-run gates are part of acceptance when prescribed by the tech design). The addenda are short on purpose; major restructuring of the existing three invariants is now listed as a sixth item under Items Explicitly Deferred or Rejected.

**Status of these resolutions.** All three comments are addressed in this revision. If the next drafting pass surfaces further altitude or placement issues — particularly around how the Control Contract addenda compose with the existing invariants in the rendered skill output — capture them as new comments rather than reopening this resolution log.

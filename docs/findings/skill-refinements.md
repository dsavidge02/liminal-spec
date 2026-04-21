# Liminal Spec Skill Refinements — Field Findings

Distilled from running `ls-tech-design-v2` and `ls-team-impl-v2` end-to-end on the streaming-control-panel project (Epic 1, Stories 0–9). Scope is refinements that apply cross-project. Windows-platform and this-machine-specific mechanics are out of scope here — those live in the project's Codex harness runbook.

**Skills impacted:** `ls-team-impl` + `ls-team-impl-v2`, `ls-tech-design` + `ls-tech-design-v2`, and the joint UI-spec surface of `ls-tech-design-v2` + `ls-team-impl-v2`.

All eight refinements below are textual changes to skill bodies or templates. None require platform or tooling work.

---

## `ls-team-impl` and `ls-team-impl-v2`

Six refinements. All apply equally to v1 and v2.

### 1. Team-spawned teammates don't receive the Agent `prompt` field

**Problem.** `Agent({team_name, name, subagent_type, prompt})` does not route `prompt` to the teammate's mailbox. The teammate wakes with an empty inbox and emits rapid-burst idle notifications.

**Evidence.** Caught on first teammate spawn of the run — 4 idle notifications in 11 seconds, no substantive message. An earlier standalone spawn (no `team_name`) had received its `prompt` at spawn, confirming team-spawn path treats it differently.

**Change.** In the story-implementation-cycle section of the skill body, add:

> After `Agent({team_name, ...})`, immediately follow with `SendMessage({to: <name>, message: <task>})` carrying the full task. The spawn-time `prompt` field is not reliably delivered for team-spawned teammates. If a newly spawned teammate emits 3+ idle notifications in quick succession without a substantive message, assume empty mailbox and deliver the task via SendMessage.

### 2. Idle notifications fire on conversation-turn boundaries, not stops

**Problem.** Teammates actively running long-lived background work emit periodic idle notifications every 2–3 minutes because each message to team-lead ends a conversation turn. Orchestrators misread these as "teammate is stopped."

**Evidence.** Across 15+ minutes of active work, a teammate emitted periodic idles while its background process continued producing hundreds of events.

**Change.** Extend the existing "Idle Notifications Are Unreliable Signals" subsection with the two failure modes:

> - *Rapid bursts immediately post-spawn.* Empty mailbox — deliver the task via SendMessage.
> - *Periodic idles during long-running work.* Conversation-turn boundaries — ignore, wait for substantive messages. Only nudge if silence exceeds a duration unreasonable for the scope of work.

### 3. "Senior-engineer subagent" language creates a standalone-spawn trap

**Problem.** The Orchestrator Role section says: *"Quick fixes (typos, one-file adjustments) → fire a `senior-engineer` subagent."* The word "subagent" reads as a *different spawn mechanism* from the team-based workflow the rest of the skill enforces. Orchestrators reach for `Agent()` without `team_name` — a standalone subagent with no mailbox, no task-list visibility, and stricter default permission posture.

**Evidence.** Caught twice on one project by user interruption. Symptoms when the trap fires: `TaskUpdate` returns `Task not found`; teammate cannot be addressed via `SendMessage`; no trace in team observability.

**Change.** Replace the quick-fixes line and add a team-discipline note:

> Quick fixes (typos, one-file adjustments) → spawn a short-lived teammate on the active team (same `team_name`, descriptive one-shot `name`). Even for 3-line edits, keep the worker on the team so task-list state, mailbox history, and permission posture stay consistent. The `senior-engineer` phrasing refers to a worker's *type*, not to a distinct spawn mechanism.
>
> **Team discipline.** Every worker spawned during the epic — implementers, reviewers, one-shot fixers — is spawned on the epic team via `Agent({team_name, name, subagent_type})`. Never spawn a standalone subagent (`Agent` without `team_name`) during orchestration.

### 4. Teammate lifecycle — silence on failure, SendMessage-to-exited-teammate

**Problem A.** Teammate templates instruct "relay the report on success." Teammates interpret this literally and stay silent when the driver exits in a failure state, even when the auto-generated report is on disk and contains the diagnosis.

**Problem B.** After a teammate exits, the team config `members[]` shrinks to just team-lead. Subsequent `SendMessage({to: <exited-name>})` silently enqueues into a nonexistent inbox — no delivery-failure signal. For mid-story fix rounds, the instinct to reuse the teammate (context already loaded) is a trap.

**Evidence A.** Driver exited with an error verdict; auto-report existed on disk; teammate made no SendMessage; team-lead discovered the failure by filesystem polling after a second idle-notification burst.

**Evidence B.** Same incident. Team-lead's subsequent SendMessage to the exited teammate silently dropped.

**Change.** In the materialized handoff template:

> Relay the impl-report to team-lead on **every** driver exit — success, failure, or unknown. Never stay silent on failure; the report is the terminating action regardless of verdict.
>
> Do not exit immediately after relaying; remain idle in case team-lead has a follow-up, until explicitly released.

And in the orchestrator's lifecycle guidance:

> Before any SendMessage to a named teammate, verify `members[]` in the team config includes the target. Prefer spawning a fresh teammate for a fix round over reusing one that may have exited.

### 5. Pre-acceptance structural gate for CLI-fabricated workarounds

**Problem.** When a CLI subagent hits an environmental limitation silently (network disabled, tool missing, permission denied), it can autonomously fabricate a local workaround that looks green on the acceptance gate. Dual review passes. The structural gate in the orchestrator is the only backstop.

**Evidence.** On this project, a CLI subagent without network access fabricated `link:` file-path dependencies pointing at arbitrary local directories outside the workspace, producing a locally-green but unportable repo. Caught by an orchestrator file-state spot-check — not by gates or review.

**Change.** Add to the orchestrator's pre-acceptance checks:

> Before accepting a story, scan for known CLI-fabrication patterns. For each CLI subagent in your toolchain, maintain a list of "impossible outputs" (e.g., `link:` paths outside the workspace, absolute host paths in configs, unresolved environment-variable literals in committed files) and grep for them as a deterministic pre-acceptance gate. This catches silent-environment-disable scenarios where the subagent worked around a limit instead of reporting it.

### 6. Delivery-assertion marker for every CLI prompt pipe

**Problem.** When a prompt file is piped to a CLI via stdin (or via file argument), path-delivery failures are silent: the CLI receives an empty file, hallucinates plausible-looking work from residual context, produces a transcript with successful-looking events, and reports completion. The repo state shows zero change, but the transcript reads as if work happened. Wastes significant tokens and orchestrator attention chasing phantom results.

**Evidence.** On this project, a path-resolution divergence caused the CLI to receive a 0-byte stdin. It produced a 558 KB, 248-event transcript with command_execution items, a turn.completed event, and a 31K-token agent_message describing 26 files it claimed to have created. `git status` showed nothing. ~25 minutes spent investigating what the CLI reported before realizing the CLI never saw the prompt.

**Change.** Adopt a universal delivery-assertion pattern for CLI subagents:

> Every prompt must open with: *"First thing: echo `<UNIQUE_MARKER>` before anything else."* After the CLI exits, the caller grep-verifies the marker appears in the transcript; `grep -c <MARKER>` must return ≥1. If 0, the prompt did not reach the CLI — stop and investigate delivery before trusting any reported results.
>
> This is four lines of orchestration that deterministically catches empty-prompt hallucinations and any future pipe/path-delivery failure on round one.

---

## `ls-tech-design` and `ls-tech-design-v2`

One refinement. Applies equally to v1 and v2.

### 7. Prescribed toolchain config snippets must be runtime-validatable

**Problem.** Tech-design documents prescribe configuration snippets (build tools, bundlers, test runners, framework configs) that are authored from the tool's written documentation but never validated against the tool's runtime requirements. Implementers copy them verbatim. Dual review passes — reviewers check faithfulness-to-spec, not tool-shape correctness. The spec itself carries the defect and it only surfaces when the tool actually runs.

**Evidence.** On Story 7 of this project, three distinct spec-shape defects were caught by the observed-run gate in a single story cycle:

- Prescribed config had an empty "reserved slot" that the build tool rejected as invalid.
- An adjacent section of the same config was missing a required `rollupOptions.input`.
- The root `package.json` was missing a required `main` field the tool needed to locate the built output.

All three invisible to dual review. All three invisible to the automated gate (format/lint/typecheck/unit tests don't exercise bundler-dev-server construction).

**Change (lighter-weight).** Add to tech-design verification:

> For every prescribed config snippet, ask: *"Would this validate as-is if the tool ran against it right now?"* Annotate each snippet with the tool's minimum-required shape at that level (e.g., "each configured section must have a rollup input"). Authors reviewing the snippet under this constraint catch the "empty slot" / "reserved for future" defects at spec time.

**Change (heavier-weight, where feasible).** When the prescribed toolchain is well-known and self-validating, add a spec-authoring step: run the tool against a synthetic repo built from the snippet and confirm it starts. Not always practical, but cheap for framework-level configs like bundlers and dev servers.

**Load-bearing corollary.** The observed-run gate (a human running the prescribed end-to-end command at least once before acceptance) is not ceremonial — it catches an entire class of defect that structural review cannot. Keep it in the acceptance path, not as an optional extra.

---

## `ls-tech-design-v2` + `ls-team-impl-v2` (joint UI-spec surface)

One refinement. V2-specific — concerns the UI-spec companion produced by v2 tech design and consumed by v2 team implementation.

### 8. UI-spec verification surface must specify tool capture mode

**Problem.** When `ls-tech-design-v2` produces a UI spec whose verification surface is a screenshot tool, neither skill specifies the capture mode. Default viewport-only captures silently truncate any content below the configured viewport height. Structural compliance checks still pass — the named components exist in code, the screenshots exist on disk, the named states are reachable. The human visual-review gate is the only backstop, and even that depends on the reviewer noticing content is missing.

**Evidence.** On this project's first UI-scoped story, 17 Playwright baselines were produced per the UI-spec state matrix. All 17 passed structural review. The footer component existed, rendered, composed in the right place — but sat below the default 1280×800 fold. All 16 default-viewport baselines clipped the footer. Caught by the human visual-review gate on the first pass, but it was a lucky catch, not a structural one.

**Change to `ls-tech-design-v2`.** In the UI-spec template's Verification Surface scaffold:

> Screenshots capture the full page (`fullPage: true` or tool-equivalent) by default. Viewport-only captures must be named explicitly in the state matrix as "above-the-fold" or "header-only" and annotated with the capture mode.
>
> For responsive-width states (e.g., `<state> at 960×600`), the dimension communicates minimum viewport target, not capture rectangle. Baselines capture full-page at the specified width, not clipped to the specified height.

**Change to `ls-team-impl-v2`.** In the UI-scoped implementer template:

> When the verification surface is screenshots and the spec doesn't specify per-state capture mode, the task section must require every capture call to pass `fullPage: true` (or tool-equivalent) plus deterministic options (animations disabled, fonts loaded).

And in the reviewer template:

> Verify every screenshot call in test files captures full-page unless the spec explicitly marks that state as viewport-only. A viewport-default capture on a full-landing baseline is a Major finding.

**General lesson.** When a spec names a verification tool without declaring its non-default capture mode, the tool's default silently narrows the surface humans can verify. The spec-authoring layer is the right place to fix it — propagating to every state matrix produced by the skill.

---

## Priority summary

| # | Refinement | Skill(s) | Confidence | Effort |
|---|------------|----------|------------|--------|
| 1 | Team-spawn SendMessage delivery | ls-team-impl(*-v2) | High | Low — skill-body text |
| 2 | Idle-notification semantics | ls-team-impl(*-v2) | High | Low — skill-body text |
| 3 | Standalone-subagent trap | ls-team-impl(*-v2) | High | Low — re-word two sentences |
| 4 | Teammate lifecycle (silence + members-shrink) | ls-team-impl(*-v2) | High | Low — template text |
| 5 | Pre-acceptance fabrication gate | ls-team-impl(*-v2) | Medium | Low — one added check |
| 6 | Delivery-assertion marker | ls-team-impl(*-v2), CLI-subagent skills | High | Low — prompt preamble + grep |
| 7 | Tech-design snippet validation | ls-tech-design(*-v2) | High | Low-to-medium — authoring step |
| 8 | UI-spec capture mode | ls-tech-design-v2 + ls-team-impl-v2 | High | Low — one-line defaults |

# Epic-Level Verification Protocol

After all stories are accepted and pre-verification cleanup is done, run this full-codebase review before shipping.

## Setup

Create a verification output directory with a subdirectory per reviewer. For example:

```
verification/
  opus/
  sonnet/
  gpt54/
  gpt53-codex/
```

## Phase 1: Four Parallel Reviews

Launch four reviewers simultaneously. Each reads the full epic, the full tech design, and the entire codebase. Each writes a detailed review report to their designated directory.

**Two Claude teammates** (general-purpose, not senior-engineer):

1. **Opus reviewer** — reads epic, tech design, all source files, all test files. Writes `epic-review.md` to their directory.
2. **Sonnet reviewer** — same artifacts, same task, writes to their directory.

**Two external model reviews** (each managed by a general-purpose teammate who loads the CLI skill):

3. **gpt-5.4 review** — the primary external verifier. Same artifacts, writes review. The teammate captures the output and writes the report file.
4. **gpt-5.3-codex review** — secondary diversity verifier, if available. Same artifacts, writes review. Same capture pattern. If gpt-5.3-codex is not available, proceed with three reviewers.

Each reviewer's prompt:

- Read the epic (path), the tech design (path), and every source and test file in the project
- Specify the exact working directory for CLI reviewers
- Do a thorough critical review of the full implementation against the epic and tech design
- Organize findings by severity (Critical, Major, Minor)
- Verify AC/TC coverage, interface compliance, architecture alignment, test quality
- Check boundary inventory: is any external dependency still a stub?
- Write the full report to their output file

**Wait for all reports before proceeding.** Do not start Phase 2 until every report is written.

## Phase 2: Meta-Reports

Send each reviewer the paths to all review reports. Each reviewer reads all reports and writes a meta-report to their directory:

- Rank the reports from best to worst
- For each report: what's good about it, what's not good about it
- Describe what they would take from each report if synthesizing a single best review

**Wait for all meta-reports before proceeding.**

## Phase 3: Orchestrator Synthesis

Read all review reports and all meta-reports. Produce a synthesized assessment:

1. **Cross-reference findings.** Build a table: which findings appear in multiple reports (high confidence), which are unique to one reviewer (investigate).
2. **Assess severity.** Claude models tend to grade generously. External models tend to grade conservatively. Apply your own judgment — don't average.
3. **Categorize the fix list:**
   - Must-fix: ship blockers
   - Should-fix: correctness or quality issues
   - Trivial: small fixes that take a few lines of code
4. **Present findings to the human.** Include the categorized list, your recommended ship-readiness grade, and any items requiring human input. Discuss. Get direction. Then dispatch.
5. **Materialize the complete fix list to a file** before constructing the handoff prompt. Include all items the human approved — must-fix, should-fix, and trivial. Do not filter out small items. Context distance causes drops; the list must exist as a readable artifact.

## Phase 4: Fixes

After discussion and human approval:

- Launch a teammate with the CLI to implement the approved fixes. Give them the fix list file, the epic, and the tech design.
- After fixes: launch a fresh review targeting the specific changes to confirm correctness.
- Run the discovered epic acceptance gate yourself to confirm all required gates pass.
- Check boundary inventory one final time: no external dependencies should remain as stubs.
- Stage, commit (`fix: epic verification fixes`), and report completion to the human.

Update log state to `COMPLETE`.

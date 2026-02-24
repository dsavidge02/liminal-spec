# Liminal Spec v0.5.0 — Story-Driven Implementation

## Context

The current pipeline (v0.4.1) generates prescriptive prompt packs from stories for agent execution. This creates time overhead, bypasses plan mode, turns engineers into "prompt paste runners," and creates role ambiguity in teams. Through conversation with the user and cross-model validation (GPT 5.3), we converged on: stories become the sole implementation artifact, with functional sections (BA/SM) and technical implementation sections (Tech Lead) in a single document. Prompt packs and execution orchestration are removed. The engineer receives a complete story and implements using their own judgment and plan mode.

## Changes Summary

## Complete Story Structure

The story is a functional acceptance artifact (orthodox Agile) with an expanded technical section below. Two distinct halves, different owners, different sign-off authorities.

```
# Story N: [Title]

─── FUNCTIONAL (BA/SM authors, PO accepts) ───

## Objective
## Scope (In/Out)
## Dependencies / Prerequisites
## Acceptance Criteria + TCs (full Given/When/Then)
## Error Paths
## Definition of Done
  - [ ] All ACs met
  - [ ] All TC conditions verified
  - [ ] PO accepts

─── TECHNICAL (Tech Lead authors, Tech Lead signs off) ───

## Technical Implementation
  ### Architecture Context
  ### Interfaces & Contracts
  ### TC → Test Mapping
  ### Risks & Constraints
  ### Spec Deviation
## Technical Checklist
  - [ ] All TCs have passing tests
  - [ ] TypeScript compiles clean
  - [ ] Lint/format passes
  - [ ] No regressions on prior stories
  - [ ] Verification: `bun run verify`
  - [ ] Spec deviations documented (if any)
```

**Ownership boundary:** BA/SM never touches below the functional DoD. Tech Lead never touches above it. The TC → test mapping in the technical section references TCs from the functional section but does not modify them.

**Sign-off:** PO accepts from the functional half. Tech Lead signs off from the technical half. The engineer serves both.

---

## Changes Summary

```
UNCHANGED:  ls-research, ls-epic (phases 1-2)
UPDATE:     ls-tech-design — strip Orchestrator/prompt references from phase + template
REWRITE:    ls-story → functional sharding only (strip prompt packs)
NEW:        ls-story-tech → Tech Lead adds technical sections to stories
REWRITE:    ls-impl → story-driven, plan-mode compatible (strip orchestration)
NEW:        story-tech-verification-prompt.md (example for ls-story-tech)
UPDATE:     tech-design-verification-prompt.md (downstream is now BA/SM + TL, not Orchestrator)
UPDATE:     verification-model.md — strip Orchestrator/prompt/dual-validator references
UPDATE:     terminology.md — update story/prompt/orchestrator definitions
SIMPLIFY:   model-selection.md
UNBUNDLE:   prompting-opus-4.6.md, prompting-gpt-5x.md (remove from manifest; keep files as reference)
UPDATE:     manifest.json, router, senior-engineer agent, build tests
UPDATE:     README.md, CLAUDE.md — align with new pipeline
```

## File-by-File Plan

### 1. `src/phases/story.md` — REWRITE

**Current:** ~660 lines. Functional sharding + prompt pack generation + orchestration.
**Target:** ~250-300 lines. Functional sharding only.

**Structure:**
```
# Story Sharding
## Purpose
## What a Story Is / What a Story Is NOT
## Story 0: Foundation Story
## Story Derivation (functional coherence, sequencing, scope)
## Story Structure (template — functional sections only)
## Why Full AC/TC Detail in Stories
## Integration Path Trace (Required)
## Coverage Gate
## Validation Before Handoff
## Output
```

**Keep:** Story derivation (lines 105-129), Integration Path Trace (lines 203-235), Story 0 guidance (lines 60-101), story structure template functional sections, full AC/TC reasoning.

**Cut entirely (~350 lines):**
- Prompt Pack Creation section (lines 238-454)
- Prompt Validation and Final Dual Verification Gate (lines 416-453)
- TDD Principles in Prompts (lines 456-481)
- Orchestration section (lines 483-537)
- Test Tracking / running totals (lines 539-556)
- Impl vs Verifier Prompts (lines 562-574)
- Prompt Writing Reference and file structure (lines 587-656)
- All "Orchestrator" role framing

**Add:**
- Coverage gate: table mapping every AC/TC to a story number. Unmapped = blocked.
- Note that technical sections will be added by Tech Lead in ls-story-tech phase.
- Updated downstream consumer: Tech Lead validates stories align with technical seams.

**Shared deps:** `confidence-chain`, `context-isolation`, `writing-style-epic`
(Remove: `model-selection`, `prompting-opus-4.6`)

---

### 2. `src/phases/story-tech.md` — NEW

**Target:** ~200-250 lines.

**Structure:**
```
# Story Technical Enrichment
## Purpose
## Role: Tech Lead as Enricher and Validator
## Inputs (functional stories + tech design + epic)
## The Technical Sections
  ### Architecture Context
  ### Interfaces & Contracts
  ### TC → Test Mapping
  ### Risks & Constraints
  ### Technical Definition of Done
  ### Spec Deviation Field
## Story Contract Requirements (the four non-negotiables)
## Story 0: Technical Enrichment
## The Consumer Gate
## Coherence Check Against Tech Design
## Common Failure Modes
## Output: Complete Stories
```

**Key content:**
- Four story contract requirements baked into the skill:
  1. TC → test mapping present (except Story 0, which has no TCs — see Story 0 section)
  2. Technical DoD present (verification commands + regression expectation)
  3. Spec deviation field present (empty if none)
  4. Technical section describes targets, not steps
- Consumer gate: "Could an engineer execute from this story without clarifying questions?"
- Coherence check: all interfaces from Tech Design are covered across stories
- Story 0 gets simplified technical sections (types to create, config to validate — no TC mapping)
- Common failure modes: steps masquerading as targets, missing TC mappings, vague DoD, rubber-stamped spec deviation field

**Shared deps:** `confidence-chain`, `verification-model`, `writing-style`

---

### 3. `src/phases/impl.md` — REWRITE

**Current:** ~792 lines. TDD execution + execution orchestration (~400 lines).
**Target:** ~300-350 lines. Story-driven, plan-mode compatible.

**Structure:**
```
# Implementation
## Purpose
## The Implementation Mindset
## Working from a Story
  ### What You Receive
  ### Plan Before You Build
## The TDD Cycle (Recommended)
  ### Skeleton Phase
  ### TDD Red Phase
  ### TDD Green Phase
  ### Self-Review Practice
  ### Gorilla Testing Phase
  ### Verify Phase
## Handling Issues
## Constraints
## Common Issues
## Output Format
```

**Keep:** Skeleton phase (lines 113-177), TDD Red (lines 180-242), TDD Green (lines 244-277), Gorilla (lines 278-314), Verify (lines 317-361), Handling Issues (lines 30-67), Constraints (lines 69-75), Common Issues (lines 362-385).

**Cut entirely (~400 lines):**
- Execution Orchestration section (lines 387-791): pipeline, dual-validator, fix cycle, agent selection, session management, human decision points, parallel pipeline, anti-patterns, execution checklist
- All "prompt pack" and "Orchestrator" references

**Add:**
- "Working from a Story" section: what the engineer receives (story with functional + technical sections), how to use reference artifacts (epic, tech design in repo)
- "Plan Before You Build" section: explicit plan-mode compatibility. Read story → form implementation plan → get approval → execute. The story is the planning input.
- Self-review as recommended practice (inline after Red and Green sections), not as orchestrated mandates
- TDD as "recommended, not mandatory" framing — the verification gate is the hard requirement, the process is the engineer's judgment

**Shared deps:** `confidence-chain`, `verification-model`, `testing`, `model-selection`
(Remove: `prompting-gpt-5x`, `prompting-opus-4.6` — implementation doesn't prescribe model choice)

---

### 4. `src/examples/story-tech-verification-prompt.md` — NEW

Pattern follows existing verification prompts (feature-verification-prompt.md, tech-design-verification-prompt.md).

**Structure:**
```
# Story Technical Enrichment Verification Prompt
## Prompt Template
  Step 1: Load skill context
  Step 2: Review files (stories, tech design, epic)
  Step 3: Evaluation Criteria
    1. Story Contract Compliance (four requirements present?)
    2. TC → Test Mapping Completeness
    3. Interface Coverage (all tech design interfaces assigned to stories?)
    4. Targets vs Steps (technical sections describe what, not how?)
    5. DoD Specificity (verification commands concrete?)
    6. Spec Deviation Accuracy (field reflects actual alignment check?)
    7. Consumer Gate (could an engineer execute?)
  Step 4: Report Format (READY/NOT READY, issues by severity)
  Step 5: Cross-Story Coverage Table
## Usage Notes
```

---

### 5. `src/examples/tech-design-verification-prompt.md` — UPDATE

**Current** references "Orchestrator" and "prompt packs" as downstream consumer (line 11: "The downstream consumer of this document is an Orchestrator who needs to derive executable stories and context-rich prompt packs").

**Change:** Update downstream consumer description to: "The downstream consumers are the BA/SM (who shard the epic into functional stories) and the Tech Lead (who adds technical implementation sections to stories)."

Update section 6 "Chunk/Work Breakdown" — reframe from "chunks become stories" to align with the story sharding model.

Update section 8 "Agent Readiness" — reframe from skeleton-phase agent readiness to engineer readiness (can an engineer plan and implement from this?).

Minor: update "Story Sharding" references to reflect the two-phase story process.

---

### 6. Shared Content Changes

**`src/shared/model-selection.md`** — SIMPLIFY
- Current: ~30 lines, prescribes specific models for specific pipeline tasks
- New: ~15 lines. General principle: different models have different strengths. Capable models work well for implementation. Detail-oriented models work well for verification. Leave specific choice to the practitioner.
- Remove: task-to-model mapping table, "Typical Flow" section, "Access Methods"

**`src/shared/prompting-opus-4.6.md`** — DROP FROM BUILD
- After manifest changes, no remaining skill depends on this file
- Keep in `src/shared/` as reference material but remove from all manifest `shared` arrays
- If a future skill needs Opus prompting guidance, it can be re-added

**`src/shared/prompting-gpt-5x.md`** — DROP FROM BUILD
- Same rationale: no remaining consumer after manifest changes
- Keep in `src/shared/` as reference material

**`src/shared/verification-model.md`** — UPDATE
- Still consumed by ls-story-tech and ls-impl (and ls-tech-design)
- Current content references Orchestrator, dual-prompt validation, prompt-specific validation patterns
- Changes needed:
  - Line 59: "Orchestrator validation" → "BA/SM + Tech Lead validation"
  - Line 72: "Prompt Validation (Multi-Agent)" section → reframe as "Story Validation"
  - Remove references to prompt packs as validation targets
  - Update "Before Stories" checkpoint to reflect two-phase story process (BA/SM then Tech Lead)
  - Update "Before Execution" checkpoint: from "prompts complete" to "stories complete (functional + technical)"
  - Keep the scrutiny gradient, multi-agent validation pattern, downstream consumer principle — all still valid

**`src/shared/terminology.md`** — UPDATE
- Line 21: "Story Sharding / Orchestration" → "Story Sharding" (Phase 4) and "Story Technical Enrichment" (Phase 4b)
- Line 22: "Creates Stories and Prompts" → "Creates functional stories from Epic"
- Add new entry: "Story Technical Enrichment" — "Tech Lead adds implementation targets, test mapping, DoD to functional stories"
- Line 32: Story definition → "A discrete, independently executable vertical slice with functional sections (BA/SM) and technical implementation sections (Tech Lead)"
- Line 33: Remove "Prompt Pack" definition entirely
- Line 66: "Dual-Validator Pattern" — keep definition but note it's optional, not a required pipeline stage
- Add: "Story Contract" — "Four non-negotiable requirements: TC→test mapping, technical DoD, spec deviation field, targets not steps"
- Add: "Consumer Gate" — "Can an engineer execute from this story without clarifying questions?"
- Remove: "Lite Mode" anti-pattern at line 91 — contradicts the new philosophy of scaling methodology to work size

---

### 6b. `src/phases/tech-design.md` — UPDATE (not rewrite)

**Current:** ~318 lines. Generally sound but references Orchestrator as downstream consumer.

**Changes (surgical, not rewrite):**
- Line 227: "the Orchestrator maps chunks to stories" → "the BA/SM maps chunks to stories during story sharding"
- Line 287: "Before handing to Orchestrator" → "Before handing to Story Sharding"
- Line 301: "The Orchestrator validates by confirming they can derive stories" → "The BA/SM validates by confirming they can derive stories from the design. The Tech Lead validates by confirming they can shard the design into story-level technical sections."

---

### 6c. `src/templates/tech-design.template.md` — UPDATE (not rewrite)

**Current:** ~773 lines. Template is mostly sound but carries prompt-era assumptions.

**Changes (surgical):**
- Line 11: "Phase Prompts | Source of specific file paths, interfaces, and test mappings" → "Story Tech Sections | Source of implementation targets, interfaces, and test mappings"
- Line 578: "quality gates that prompts reference" → "quality gates that story technical sections reference"
- Line 699: "the Orchestrator validates by confirming they can derive stories from this design" → "the BA/SM validates by confirming they can shard stories, and the Tech Lead validates by confirming they can create story technical sections"
- Line 770: "Story Prompts: `stories/`" → "Stories: `stories/`"

---

### 6d. `README.md` — UPDATE

**Changes:**
- Phase table (line 17-24): Update Phase 4 out from "Stories + Prompt Packs" to "Functional Stories". Add Phase 4b row: "Story Tech | Stories (functional) + Tech Design | Complete Stories". Update Phase 5 from "Prompts → Verified code" to "Complete Stories → Verified code"
- Line 39: Remove "Prompt contract" key idea. Replace with "Story as implementation artifact" — complete stories with functional and technical sections are the sole handoff to engineers
- Lines 61, 93-94: Update `/ls-story` description from "shard into stories and generate prompt packs" to "shard epic into functional stories". Add `/ls-story-tech` row.
- Lines 96-105: Rewrite "Execution SOP" section — remove prompt-based flow. Replace with: "Stories contain functional requirements (ACs, TCs) and technical implementation sections (targets, test mapping, DoD). Engineers implement from complete stories using TDD discipline."
- Line 80: Update skill pack directory listing to include `04b-story-technical-enrichment/`
- Line 93-94: Update markdown pack table to include the new file

---

### 6e. `CLAUDE.md` — UPDATE

**Changes:**
- Plugin structure section (~line 27): Add `ls-story-tech/SKILL.md` to the skills listing
- Skills table (~line 93): Add ls-story-tech row, update ls-story and ls-impl descriptions
- Any references to "prompt packs" or "orchestrator" → update to story-driven terminology
- Update "What Gets Built" table to include the new skill

---

### 7. `manifest.json` — UPDATE

```json
{
  "version": "0.5.0",
  "skills": {
    "ls-research": { /* NO CHANGE */ },
    "ls-epic": { /* NO CHANGE */ },
    "ls-tech-design": { /* NO CHANGE */ },
    "ls-story": {
      "name": "ls-story",
      "description": "Break an epic into functional stories with full acceptance criteria and test conditions. Validates coverage completeness and cross-story integration path coherence.",
      "phases": ["story"],
      "shared": ["confidence-chain", "context-isolation", "writing-style-epic"]
    },
    "ls-story-tech": {
      "name": "ls-story-tech",
      "description": "Add technical implementation sections to functional stories. Tech Lead maps ACs to implementation targets, test strategies, and verification criteria. Validates via consumer gate.",
      "phases": ["story-tech"],
      "shared": ["confidence-chain", "verification-model", "writing-style"],
      "examples": ["story-tech-verification-prompt"]
    },
    "ls-impl": {
      "name": "ls-impl",
      "description": "Implement stories using TDD discipline. Plan-mode compatible. Reads complete stories with functional and technical sections.",
      "phases": ["impl"],
      "shared": ["confidence-chain", "verification-model", "testing", "model-selection"]
    }
  }
}
```

Key changes:
- ls-story: remove `model-selection`, `prompting-opus-4.6`; add `context-isolation`, `writing-style-epic`
- ls-story-tech: new entry with `confidence-chain`, `verification-model`, `writing-style`, example
- ls-impl: remove `prompting-opus-4.6`, `prompting-gpt-5x`; add `testing`; keep `model-selection`

---

### 8. `src/commands/liminal-spec.md` — UPDATE

Add Phase 4b to the phase table:

```
| 4. Story Sharding   | /ls-story       | Epic + design done, ready for functional stories    |
| 4b. Story Tech      | /ls-story-tech  | Functional stories ready for technical enrichment   |
```

Add routing entry for Phase 4b → `liminal-spec:ls-story-tech`

Update Phase 4 description (remove "prompt packs"), Phase 5 description (remove "prompts").

---

### 9. `src/agents/senior-engineer.md` — MINOR UPDATE

- Update model reference from "Opus 4.5" to "Opus 4.6"
- No structural changes needed (agent already focuses on TDD/quality, not orchestration)

---

### 10. `scripts/build.ts` — UPDATE

Add to `STANDALONE_NAMES` map:

```typescript
"ls-story-tech": "04b-story-technical-enrichment",
```

No other build.ts changes — the composition is manifest-driven and handles new skills automatically.

---

### 11. `scripts/__tests__/build.test.ts` — UPDATE

**Add `"ls-story-tech"` to all expected skill arrays** (lines 85-91, 172-178).

**Add to build summary test** (line 66-72):
```typescript
expect(buildOutput).toContain("skill: ls-story-tech");
```

**Update impl content test** (lines 213-219):
- Remove: `expect(content).toContain("Dual-Validator Pattern")`
- Add: `expect(content).toContain("Plan Before You Build")`
- Keep: `expect(content).toContain("NotImplementedError")` or similar TDD content

**Add ls-story negative regression test:**
```typescript
test("story does not contain removed prompt-pack content", async () => {
  const content = await Bun.file(
    join(DIST_PLUGIN, "skills", "ls-story", "SKILL.md")
  ).text();
  expect(content).not.toContain("Prompt Pack");
  expect(content).not.toContain("Orchestrator");
});
```

**Add story-tech content test:**
```typescript
test("story-tech contains contract requirements", async () => {
  // verify Story Contract Requirements and Consumer Gate present
});
```

**Add to standalone expectedMdFiles** (lines 245-251):
```
"04b-story-technical-enrichment-skill.md"
```

**Add to marketplace expectedPaths** (lines 147-152):
```
join(MARKETPLACE_PLUGIN, "skills", "ls-story-tech", "SKILL.md")
```

**Add to source safety criticalFiles** (lines 312-322):
```
"src/phases/story-tech.md"
```

---

### 12. Version & Release

**Version bump in 4 places:** `version.txt`, `manifest.json`, `package.json`, `.claude-plugin/marketplace.json` → `0.5.0`

**Changelog:** Add v0.5.0 entry at top of CHANGELOG.md with Added/Changed/Removed/Migration sections.

---

## Execution Order

Dependencies determine the sequence:

1. **Content — core skill changes (parallelizable):**
   - Rewrite `src/phases/story.md`
   - Create `src/phases/story-tech.md`
   - Rewrite `src/phases/impl.md`
   - Create `src/examples/story-tech-verification-prompt.md`

2. **Content — ripple effect updates (after reviewing step 1 output):**
   - Update `src/shared/terminology.md` FIRST (establishes canonical vocabulary for all other edits)
   - Then parallelizable:
     - Update `src/phases/tech-design.md` (strip Orchestrator references)
     - Update `src/templates/tech-design.template.md` (strip prompt-era assumptions)
     - Update `src/examples/tech-design-verification-prompt.md`
     - Update `src/shared/verification-model.md` (strip Orchestrator/prompt references)
     - Simplify `src/shared/model-selection.md`
     - Remove `prompting-opus-4.6.md` and `prompting-gpt-5x.md` from manifest (keep files as reference)

3. **Configuration (depends on content existing):**
   - Update `manifest.json` (new skill + updated deps + version + drop unused shared refs)
   - Update `src/commands/liminal-spec.md` (router)
   - Update `src/agents/senior-engineer.md` (minor)
   - Update `scripts/build.ts` (standalone name)

4. **Tests (depends on config):**
   - Update `scripts/__tests__/build.test.ts`

5. **Documentation and release (last):**
   - Update `README.md` (pipeline, commands, SOP)
   - Update `CLAUDE.md` (plugin structure, skills table)
   - Bump version in 4 files
   - Write changelog entry

## Verification

```bash
bun run build       # Compose all skills — verify 6 skills output (was 5)
bun run validate    # Structural validation passes
bun test            # Integration tests pass
bun run verify      # All three in sequence

# Manual verification:
claude --plugin-dir ./dist/plugin   # Test router, new skills
# Spot-check dist/plugin/skills/ls-story-tech/SKILL.md for composition coherence
# Spot-check dist/standalone/04b-story-technical-enrichment-skill.md for standalone usability
# Verify dist/standalone/ has 6 markdown files + 2 packs
```

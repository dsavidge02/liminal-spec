# Story Technical Enrichment Verification Prompt

Use this prompt template to have an agent critically review technically enriched stories before handing off to Implementation (Phase 5).

---

## Prompt Template

**Critical Review: [Feature Name] Story Technical Enrichment**

You are reviewing technically enriched stories for [brief description]. This is Phase 4b (Story Technical Enrichment) of a Liminal Spec pipeline. The downstream consumer is an engineer who needs to implement from these stories using TDD discipline and plan mode.

**Step 1: Load liminal-spec Skill Context**

Read these files to understand the methodology and evaluation criteria:

1. **Core methodology:** `~/.claude/skills/liminal-spec/SKILL.md`
2. **Story technical enrichment guidance:** the Story Technical Enrichment section inside `~/.claude/skills/liminal-spec/SKILL.md`
3. **Implementation guidance:** the Implementation section inside `~/.claude/skills/liminal-spec/SKILL.md`

**Step 2: Review These Files**

1. **Stories (primary):** `[path to stories directory]`
2. **Tech Design (for alignment):** `[path to tech-design.md]`
3. **Epic (for AC/TC completeness):** `[path to epic.md]`

**Step 3: Evaluation Criteria**

Assess each technically enriched story against these criteria:

1. **Story Contract Compliance**
   - Does every story have all four required elements? (TC-to-test mapping, technical DoD, spec deviation field, targets-not-steps)
   - Is Story 0 appropriately simplified (no TC mapping, types/config focus)?
   - Are technical sections below the functional DoD boundary?

2. **TC to Test Mapping Completeness**
   - Does every TC in the story's functional section have a corresponding entry in the TC-to-test mapping table?
   - Are test approaches appropriate for what's being verified (service mock vs integration, assertion strategy)?
   - Are test file names specific and consistent with the tech design?

3. **Interface Coverage**
   - Are all interfaces from the tech design's Low Altitude section assigned to at least one story's Interfaces & Contracts section?
   - Are all modules from the tech design's Module Responsibility Matrix represented across story Architecture Context sections?
   - Any tech design interfaces with no story assignment?

4. **Targets vs Steps**
   - Do technical sections describe what to build (modules, interfaces, contracts) rather than how to build it (step-by-step instructions)?
   - Does the Architecture Context name modules and flows without prescribing implementation sequence?
   - Could an engineer reasonably choose a different implementation order and still satisfy the story?

5. **DoD Specificity**
   - Are verification commands concrete (`bun run verify`, not "run tests")?
   - Does the technical checklist include regression expectations?
   - Is spec deviation documentation included as a checklist item?

6. **Spec Deviation Accuracy**
   - Does the spec deviation field reference specific tech design sections that were checked?
   - When deviations exist, is the rationale clear and the scope bounded?
   - Are "None." entries credible (do they appear to reflect actual checking, not rubber-stamping)?

7. **Consumer Gate**
   - Could an engineer implement from this story without asking clarifying questions?
   - Is the Architecture Context sufficient to orient an engineer in the codebase?
   - Is the TC-to-test mapping sufficient to write tests without deriving approaches from scratch?

**Step 4: Report Format**

Provide your review in this structure:

```
## Overall Assessment
[READY / NOT READY] for Implementation

## Strengths
[What the technical enrichment does well]

## Issues

### Critical (Must fix before Implementation)
[Issues that would block an engineer from implementing]

### Major (Should fix)
[Issues that would cause confusion or rework during implementation]

### Minor (Fix before handoff)
[Polish items -- address these too, not just blockers]

## Missing Elements
[Anything that should be present but isn't]

## Cross-Story Gaps
[Interface or module coverage gaps across the story set]

## Recommendations
[Specific fixes, in priority order]

## Questions for the Tech Lead
[Clarifying questions that would improve the technical sections]
```

Be thorough and critical. The goal is to catch issues before they surface during implementation, where they're more expensive to fix.

**Step 5: Cross-Story Coverage Table**

As part of your review, produce a coverage table mapping tech design elements to stories:

```
| Tech Design Element | Type | Assigned Story | Coverage Notes |
|---|---|---|---|
| SessionService | Module | Story 1, Story 2 | Created in S1, extended in S2 |
| AuthMiddleware | Module | Story 2 | |
| Session type | Interface | Story 0 | Foundation |
| TC-1.1a | Test Mapping | Story 1 | Service mock approach |
| TC-2.1a | Test Mapping | Story 2 | Missing test file assignment |
```

This table makes gaps immediately visible. If a tech design element has no story assignment, or a TC has no test mapping, flag it.

---

## Usage Notes

- Run this with a verification-oriented model (detail-oriented models recommended for thoroughness)
- Can also run with multiple agents in parallel for diverse perspectives
- Compare the cross-story coverage table against the epic's AC/TC table for full chain coverage
- Critical and Major issues should be addressed before Implementation handoff
- The engineer will also validate implicitly during implementation -- if they can't implement from the story, it goes back

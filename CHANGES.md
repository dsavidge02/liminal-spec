# Pending Changes (next release)

Scratch pad for tracking changes as we go. Will be consolidated into CHANGELOG.md at release time.

---

### Stack-neutral data contracts (214114b)

**What changed:** Data contracts in epics and stories now use documentation tables (endpoints, field/type/validation tables, error response tables) instead of language-specific syntax (TypeScript interfaces, Zod schemas). Contracts scoped to significant system boundaries (frontend-to-backend, app-to-external) rather than internal layer contracts.

**Files touched:**
- `src/phases/epic.md` — "When Technical Detail Is Appropriate" reframed, Data Contracts section replaced with doc-table example, template updated, validation gates updated
- `src/phases/story-simple.md` — Same reframe, contract example replaced (now includes success response), validation gates updated
- `src/phases/publish-epic.md` — Removed TypeScript references in story structure, "removed/transformed" lists, validation checklist
- `src/phases/team-spec.md` — Updated business epic validation wording
- `src/templates/feature-spec.template.md` — Updated response types placeholder and validation checklist (not in build, consistency only)

**Why:** Epics are functional, stack-neutral documents. Code syntax (TypeScript interfaces, Zod schemas) is an implementation opinion that belongs in Tech Design. Documentation tables capture the same precision (field names, types, cardinality, validation rules) without locking into a language.

---

### Epic size and scope check (1049aac)

**What changed:** Added a scope check checkpoint after the Epic Structure section. When the agent estimates an epic will exceed ~800 lines (5+ major flows or 30+ expected ACs), it informs the user and offers to analyze requirements for natural splitting points (workflow boundaries, system boundaries, phased delivery). User can proceed with a single large epic if preferred — checkpoint, not a gate.

**Files touched:**
- `src/phases/epic.md` — New "Epic Size and Scope Check" subsection under "The Epic Structure"

**Why:** Large epics create downstream pressure — tech design consumes the full epic as input context, validation quality degrades with length, and internal consistency becomes harder to maintain. Catching this early gives the user the option to split before sinking effort into a massive single artifact.

---

### Individual story files + optional business epic (6002d5f)

**What changed:** `ls-publish-epic` now writes each story to its own numbered file in a `stories/` folder (`00-foundation.md`, `01-[name].md`, etc.) instead of one monolithic story file. The business epic is now optional — the skill presents the user with the choice (stories only, or stories + business epic) before starting work.

**Files touched:**
- `src/phases/publish-epic.md` — New "What This Produces" section with user choice, Step 1 writes individual files, Step 2 conditional on user choice, validation split into story-only and business-epic items, all references updated for individual files
- `src/phases/team-spec.md` — Updated all publish-epic references throughout drafting, verification, handoff, and final verification phases for individual story files and optional business epic
- `manifest.json` — Updated skill and individual plugin descriptions
- `scripts/__tests__/build.test.ts` — Updated content assertion for renamed heading

**Why:** Monolithic story files required a separate step to break into individual files and re-validate. Individual files from the start eliminates that rework. Business epic is not always needed — many workflows go straight from stories to implementation without a PO-facing roll-up.

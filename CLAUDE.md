# Liminal Spec

## What This Is

A spec-driven development methodology packaged as a **skill pack**. The pack contains 10 stable self-contained skills — 5 for the full pipeline, 1 current-docs continuity skill, and 4 team orchestration skills — plus experimental skills (currently `ls-tech-design-v2` and `ls-team-impl-v2`) that pilot proposed methodology changes.

This is NOT a library or npm package. The build output is markdown files organized as installable skill directories. There are two distribution channels:

**Skill Pack** (`dist/skills/`) — Installable skill directories. Each contains a `SKILL.md` with YAML frontmatter. Copy to `~/.claude/skills/` or `~/.agents/skills/`.

**Standalone Markdown** (`dist/standalone/`) — Per-skill markdown files for paste-into-chat use, plus two release packs: `liminal-spec-skill-pack.zip` (skill directories) and `liminal-spec-markdown-pack.zip` (paste-ready markdown files).

Both outputs are composed from the same source files. The build handles the packaging.

### Output Structure

```
dist/
  skills/
    ls-prd/SKILL.md            -- PRD (compressed proto-epics)
    ls-arch/SKILL.md            -- Technical Architecture
    ls-epic/SKILL.md            -- Epic (Phase 1)
    ls-tech-design/SKILL.md     -- Tech Design (Phase 2)
    ls-tech-design-v2/SKILL.md  -- (experimental) Tech Design + optional UI companion
    ls-publish-epic/SKILL.md    -- Publish Epic (Phase 3)
    ls-codex-impl/SKILL.md      -- Codex Implementation Orchestration
    ls-current-docs/SKILL.md   -- Current Docs Baseline
    ls-current-docs/references/ -- Installed-skill companion guides for functional + technical + code-map drafting
    ls-team-spec/SKILL.md       -- Team: Spec Pipeline Orchestration
    ls-team-impl/SKILL.md       -- Team: Implementation with CLI
    ls-team-impl-v2/SKILL.md    -- (experimental) Team: Implementation with CLI + UI spec consumer
    ls-team-impl-cc/SKILL.md    -- Team: Implementation with Claude Code teams
  standalone/
    *-skill.md                  -- Per-skill paste-ready markdown
    liminal-spec-skill-pack.zip
    liminal-spec-markdown-pack.zip
```

Most skills are self-contained in `SKILL.md` with shared concepts inlined by the build. `ls-current-docs` also bundles focused companion reference docs for installed-skill use while keeping the main `SKILL.md` sufficient on its own.

## Project

Source-based skill with build composition. Edit in `src/`, never in `dist/`.

### Structure

```
src/
  phases/          -- Phase-specific content (one per skill: prd, arch, epic, tech-design, publish-epic, current-state, team-impl, team-impl-cc, team-spec)
  references/      -- Bundled companion guides for installed-skill-first workflows
  shared/          -- Cross-cutting concepts inlined into multiple skills by the build
  templates/       -- Artifact templates (tech design)
  examples/        -- Verification prompt templates
scripts/
  build.ts         -- Compose src/ into dist/
  validate.ts      -- Validate dist/ output
manifest.json      -- Maps which shared files each phase skill needs
docs/              -- Reference material not yet in the build pipeline
dist/              -- Build output (gitignored)
  skills/          -- Installable skill directories
  standalone/      -- Paste-ready MDs + skill-pack and markdown-pack zips
```

### Commands

```bash
bun run build       # Compose source into dist/
bun run validate    # Validate dist/ output
bun run check       # Build + validate
bun test            # Run integration tests
bun run verify      # Build + validate + tests
```

### How the Build Works

`manifest.json` declares which shared files each phase skill needs, and may also bundle companion references for installed-skill-first workflows. `scripts/build.ts` reads the manifest, concatenates phase content + shared content in declared order, wraps with SKILL.md YAML frontmatter, and outputs to `dist/skills/<name>/SKILL.md`. When a skill declares bundled references, the build copies them into `dist/skills/<name>/references/`. It also strips frontmatter and outputs per-phase markdown files (`dist/standalone/*-skill.md`) plus pack zips for release distribution.

### What Gets Built

**Skills** (10 stable + 2 experimental — 5 full pipeline, 1 current-docs continuity, 4 team orchestration, plus the v2 pilot pair):

| Skill | Phase | Primary source | Shared dependencies |
|-------|-------|----------------|---------------------|
| `ls-prd` | 0a | `src/phases/prd.md` | confidence-chain, writing-style-epic |
| `ls-arch` | 0b | `src/phases/arch.md` | (none — self-contained) |
| `ls-epic` | 1 | `src/phases/epic.md` | confidence-chain, context-isolation, writing-style-epic |
| `ls-tech-design` | 2 | `src/phases/tech-design.md` | confidence-chain, verification-model, writing-style, testing |
| `ls-tech-design-v2` *(experimental)* | 2 | `src/phases/tech-design-v2.md` | confidence-chain, dimensional-reasoning, verification-model, writing-style, testing |
| `ls-publish-epic` | 3 | `src/phases/publish-epic.md` | confidence-chain, writing-style-epic |
| `ls-current-docs` | Current | `src/phases/current-state.md` | confidence-chain, context-isolation, dimensional-reasoning |
| `ls-codex-impl` | Team | `src/phases/codex-impl.md` | (none — self-contained) |
| `ls-team-impl` | Team | `src/phases/team-impl.md` | (none — self-contained) |
| `ls-team-impl-v2` *(experimental)* | Team | `src/phases/team-impl-v2.md` | (none — self-contained) |
| `ls-team-impl-cc` | Team | `src/phases/team-impl-cc.md` | (none — self-contained) |
| `ls-team-spec` | Team | `src/phases/team-spec.md` | (none — self-contained) |

**Experimental skills** (`ls-tech-design-v2`, `ls-team-impl-v2`) are pilot skills running the UI-companion experiment. See `docs/research/ls-tech-design-v2-plan.md` for the experiment plan, decision matrix, and exit criteria. They are exempt from the standard skill-addition documentation rules — see "Experimental Skill Convention" below.

## How to Update Content

### Editing a phase

1. Edit the source file in `src/phases/<name>.md`
2. `bun run check` to rebuild and validate
3. Commit and push

### Editing a shared concept

Shared files affect multiple skills. Before editing, check which skills depend on it:

```bash
# Example: which skills use the writing-style shared file?
grep "writing-style" manifest.json
```

After editing, rebuild and review the affected skill outputs to make sure the inlined content reads correctly in each context. The same shared section may appear after different phase content in each skill -- verify it flows naturally in all of them.

### Adding a new shared file

1. Create the file in `src/shared/<name>.md`
2. Add it to the relevant skill(s) in `manifest.json` under the `shared` array
3. `bun run check` to rebuild and validate

### Adding content to a phase that's already inline in the source

Some content (like the epic template) is already embedded in the phase file rather than composed from a separate source. The `templates` and `examples` entries in `manifest.json` are for content that lives in separate files and gets appended. If the content is already in the phase file, don't also add it to the manifest -- that causes duplication.

### Testing locally

Copy a built skill to your skills directory:

```bash
bun run build
cp -r dist/skills/ls-epic ~/.claude/skills/ls-epic
```

### Adding a new skill

1. Create the phase source file: `src/phases/<name>.md`
2. Add the skill entry to `manifest.json` with name, description, phases, and shared dependencies
3. Update the version assertion in `scripts/__tests__/build.test.ts` if needed
4. `bun run check` to verify the build picks it up

## PR Checklist

Before opening or updating a PR:

1. Source-only edits: no manual changes in `dist/`.
2. Local gate passes: `bun run verify`.
3. For content/methodology edits:
   - spot-check affected `dist/skills/*/SKILL.md` for composition coherence
   - spot-check affected `dist/standalone/*-skill.md` for standalone usability
   - confirm standalone packs exist and no legacy `.skill` files are emitted
4. For skill additions or removals:
   - update `manifest.json` (add/remove entry)
   - update `scripts/build.ts` standalone name mapping
   - update `scripts/__tests__/build.test.ts` (expected skills, content tests, standalone files, removed-skills assertions)
   - update `CLAUDE.md` (output structure, phases list, skill table)
   - update `README.md` (skill table, pipeline tables)
   - update `src/README-pack.md` and `src/README-markdown-pack.md` (skill listings)
   - update any cross-references in other phase source files (e.g. `team-spec.md` references implementation skills)

### Experimental Skill Convention

Skills marked as experimental in the skill table (currently `ls-tech-design-v2`, `ls-team-impl-v2`) are **exempt from the public-docs portion** of the standard skill-addition rules. They are listed in `CLAUDE.md` and composed through the normal build, but they are **not** added to `README.md`, `src/README-pack.md`, or `src/README-markdown-pack.md`. The public surface stays stable while experiments run.

Rationale: experimental skills exist to test whether a methodology change earns its place. Advertising them in public docs before the experiment resolves would either overstate their status (if they retire) or create churn (if they get renamed, merged, or promoted). Holding the public surface steady lets experiments resolve cleanly.

When an experiment resolves:

- **Promote** (experimental skill graduates to a top-level skill in its own right) → update all public docs per the standard skill-addition rules at that point.
- **Merge** (experimental skill folds into its stable counterpart) → remove the experimental entry from `manifest.json`, `CLAUDE.md`, `scripts/build.ts`, and tests; fold the new content into the stable skill's sources and update the public docs to reflect the stable skill's expanded capability.
- **Retire** → remove the experimental entry everywhere (manifest, build, tests, `CLAUDE.md`); document the negative result in `docs/research/`. Public docs were never updated, so nothing to roll back there.

### About `docs/`

The `docs/` directory holds long-form reference material that may or may not be composed into builds.

## Release Process

1. Push to main (directly or via PR merge).
2. When ready to release, update version in three places:
   - `version.txt`
   - `manifest.json`
   - `package.json`
3. Update the changelog: add entry for `vX.Y.Z (YYYY-MM-DD)`.
4. Update the pack READMEs (bundled into release artifacts):
   - `src/README-pack.md` — version in title, add changelog entry
   - `src/README-markdown-pack.md` — version in title, add changelog entry
5. Run `bun run verify` (builds, validates, runs tests).
6. Commit the version bump + changelog, then tag and push: `git tag vX.Y.Z && git push origin vX.Y.Z`
7. Tag triggers release workflow -> builds, validates, tests, packages skill-pack + markdown-pack zips, creates GitHub Release with artifacts

The tag is the explicit "ship it" signal. Code can accumulate on main across multiple pushes without releasing.

## Development Principles

This project IS the liminal-spec methodology. These principles govern how you work on it:

**Progressive depth, not flat lists.** Prose establishes branches (context, why it matters). Bullets hang leaves (specifics). A section of equal-weight bullets with no framing paragraph is a sign something is wrong. Earn complexity -- don't front-load it.

**Upstream gets more scrutiny.** Epic content is the most critical -- errors cascade through every downstream phase. Downstream content is more localized. Weight your care accordingly when editing.

**Prescribe the what, not the how.** The skill should say "verify test immutability at Green exit" not "run `git diff --name-only HEAD~1 | grep test`". Projects vary. Methodology guidance stays at the principle level; implementation details belong in project-specific prompts.

**Confidence chain: AC -> TC -> Test -> Implementation.** Every piece of guidance should trace back to why it exists. If you can't articulate what failure mode a section prevents, it probably doesn't belong.

**Context isolation is structural, not aspirational.** The build composes self-contained skills precisely so consumers don't need progressive disclosure. Source files can reference each other freely. Output files must stand alone.

**Don't add without approval.** New sections, new shared concepts, new checklist items -- propose them, don't add them. The owner curates what's in the methodology.

**No content duplication in source.** Shared concepts live in `src/shared/` and get inlined by the build. If the same concept appears in two phase files, extract it to shared.

**Templates and examples are source artifacts.** They get composed into skills or bundled alongside them. Edit in `src/templates/` and `src/examples/`.

## Coherence Checks Before Release

Before tagging a release, verify content coherence:

1. `bun run verify` passes (build + validate + test)
2. Review each `dist/skills/*/SKILL.md` -- does the inlined shared content flow naturally after the phase content?
3. For significant content changes: run a cross-model comparison (e.g. Codex at high reasoning) against previous release output to catch unintended drift
4. Spot-check a `dist/standalone/*.md` file -- is it usable when pasted into a chat with no other context?

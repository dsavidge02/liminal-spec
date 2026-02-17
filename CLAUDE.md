# Liminal Spec

## What This Is

A spec-driven development methodology packaged as a **Claude Code plugin**. The plugin contains 4 self-contained skills (one per methodology phase), a router command, and a senior-engineer agent.

This is NOT a library or npm package. The build output is markdown files organized into a Claude Code plugin structure. There are three distribution channels:

**Plugin** (`dist/plugin/`) — For Claude Code users (developers, senior engineers). They install via marketplace and get slash commands (`/liminal-spec`, `/liminal-spec:epic`, etc.). The plugin bundles skills + agents + commands + marketplace metadata per the [Claude Code plugin spec](https://code.claude.com/docs/en/plugins).

**Marketplace install source** (`plugins/liminal-spec/`) — Committed, installable plugin layout used by `/plugin install ...@liminal-plugins`. This directory is generated from `dist/plugin/` by the build.

**Standalone** (`dist/standalone/`) — For non-Claude-Code users (BA, PO) who paste a single markdown file into Claude Enterprise Chat or any AI assistant. Each file is fully self-contained — no plugin infrastructure, no frontmatter, just the methodology content for one phase.

Both outputs are composed from the same source files. The build handles the packaging differences.

### Plugin Structure (what gets installed)

```
.claude-plugin/
  plugin.json              — Plugin metadata (name, version, author)
  marketplace.json         — Marketplace catalog for /plugin marketplace add
skills/
  epic/SKILL.md            — /liminal-spec:epic (Phase 2: Epic)
  tech-design/SKILL.md     — /liminal-spec:tech-design (Phase 3)
  story/SKILL.md           — /liminal-spec:story (Phase 4: Story Sharding)
  impl/SKILL.md            — /liminal-spec:impl (Phase 5: Execution)
commands/
  liminal-spec.md          — /liminal-spec (router: presents phase menu, invokes skill)
agents/
  senior-engineer.md       — Senior engineer agent for TDD implementation
```

Each skill SKILL.md has YAML frontmatter (`name` + `description`) and is self-contained — all shared concepts (confidence chain, writing style, etc.) are inlined by the build. No progressive disclosure, no reference file loading needed.

## Project

Source-based skill with build composition. Edit in `src/`, never in `dist/`.

### Structure

```
src/
  phases/          — Phase-specific content (one per skill: epic, tech-design, story, impl)
  shared/          — Cross-cutting concepts inlined into multiple skills by the build
  templates/       — Artifact templates (tech design, epic)
  examples/        — Verification prompt templates
  commands/        — Plugin command (/liminal-spec router)
  agents/          — Plugin agents (senior-engineer)
scripts/
  build.ts         — Compose src/ into dist/
  validate.ts      — Validate dist/ output
manifest.json      — Maps which shared files each phase skill needs
docs/              — Reference material not yet in the build pipeline
dist/              — Build output (gitignored)
  plugin/          — Claude Code plugin (skills/ + agents/ + commands/ + marketplace)
  standalone/      — Paste-ready MDs for non-Claude-Code users (BA/PO)
plugins/           — Committed marketplace-installable plugin directories
```

### Commands

```bash
bun run build       # Compose source into dist/
bun run validate    # Validate dist/ output
bun run check       # Build + validate
bun test            # Run integration tests
```

### How the Build Works

`manifest.json` declares which shared files each phase skill needs. `scripts/build.ts` reads the manifest, concatenates phase content + shared content in declared order, wraps with SKILL.md YAML frontmatter, and outputs to `dist/plugin/skills/<name>/SKILL.md`. It also strips frontmatter and outputs to `dist/standalone/liminal-<name>.md` for paste-into-chat distribution.

The build also copies agents, commands, generates plugin.json + marketplace.json in `dist/plugin/.claude-plugin/`, then syncs a committed marketplace install source at `plugins/liminal-spec/`.

### What Gets Built

**Command** (router — not a skill):

| Command | Source | Purpose |
|---------|--------|---------|
| `/liminal-spec` | `src/commands/liminal-spec.md` | Presents phase menu, invokes the appropriate skill |

**Skills** (4 self-contained phase skills):

| Skill | Phase | Primary source | Shared dependencies |
|-------|-------|----------------|---------------------|
| `/liminal-spec:epic` | 2 | `src/phases/epic.md` | confidence-chain, context-isolation, writing-style |
| `/liminal-spec:tech-design` | 3 | `src/phases/tech-design.md` | confidence-chain, verification-model, writing-style, testing |
| `/liminal-spec:story` | 4 | `src/phases/story.md` | confidence-chain, model-selection, prompting-opus-4.6 |
| `/liminal-spec:impl` | 5 | `src/phases/impl.md` | confidence-chain, model-selection, verification-model, prompting-gpt-5x, prompting-opus-4.6 |

**Agent:**

| Agent | Source | Purpose |
|-------|--------|---------|
| senior-engineer | `src/agents/senior-engineer.md` | TDD implementation with quality gates |

## How to Update Content

### Editing a phase

1. Edit the source file in `src/phases/<name>.md`
2. `bun run check` to rebuild and validate
3. `claude --plugin-dir ./dist/plugin` to test locally
4. Commit and push

### Editing a shared concept

Shared files affect multiple skills. Before editing, check which skills depend on it:

```bash
# Example: which skills use the writing-style shared file?
grep "writing-style" manifest.json
```

After editing, rebuild and review the affected skill outputs to make sure the inlined content reads correctly in each context. The same shared section may appear after different phase content in each skill — verify it flows naturally in all of them.

### Adding a new shared file

1. Create the file in `src/shared/<name>.md`
2. Add it to the relevant skill(s) in `manifest.json` under the `shared` array
3. `bun run check` to rebuild and validate

### Adding content to a phase that's already inline in the source

Some content (like the epic template) is already embedded in the phase file rather than composed from a separate source. The `templates` and `examples` entries in `manifest.json` are for content that lives in separate files and gets appended. If the content is already in the phase file, don't also add it to the manifest — that causes duplication.

### Testing locally

```bash
claude --plugin-dir ./dist/plugin
```

This loads the plugin from the build output. Test the router (`/liminal-spec`) and individual skills (`/liminal-spec:epic`, etc.).

### About `docs/`

The `docs/` directory holds reference material not yet incorporated into the build pipeline. Currently contains `product-research.md` (Phase 1 content, deferred). When Phase 1 is added as a skill, this content would move to `src/phases/` and be added to the manifest. Do not add docs/ content to the build without explicit approval.

### Adding a new skill

1. Create the phase source file: `src/phases/<name>.md`
2. Add the skill entry to `manifest.json` with name, description, phases, and shared dependencies
3. Update the router command (`src/commands/liminal-spec.md`) to include the new phase in the table and routing logic
4. Update the version assertion in `scripts/__tests__/build.test.ts` if needed
5. `bun run check` to verify the build picks it up
6. Test locally with `claude --plugin-dir ./dist/plugin`

## Release Process

1. Push to main (directly or via PR merge) → CI runs build + validate + test
2. When ready to release, update version in all five places:
   - `version.txt`
   - `manifest.json`
   - `package.json`
   - `.claude-plugin/marketplace.json`
   - `scripts/__tests__/build.test.ts` (version assertion in the plugin.json test)
3. Run `bun run build` to sync `plugins/liminal-spec/` from source.
4. Commit the version bump, then tag and push: `git tag vX.Y.Z && git push origin vX.Y.Z`
5. Tag triggers release workflow → builds, validates, packages plugin + standalone zips, creates GitHub Release with artifacts

The tag is the explicit "ship it" signal. Code can accumulate on main across multiple pushes without releasing.

## Development Principles

This project IS the liminal-spec methodology. These principles govern how you work on it:

**Progressive depth, not flat lists.** Prose establishes branches (context, why it matters). Bullets hang leaves (specifics). A section of equal-weight bullets with no framing paragraph is a sign something is wrong. Earn complexity — don't front-load it.

**Upstream gets more scrutiny.** Phase 2 content is the most critical — errors cascade through every downstream phase. Phase 5 content is more localized. Weight your care accordingly when editing.

**Prescribe the what, not the how.** The skill should say "verify test immutability at Green exit" not "run `git diff --name-only HEAD~1 | grep test`". Projects vary. Methodology guidance stays at the principle level; implementation details belong in project-specific prompts.

**Confidence chain: AC → TC → Test → Implementation.** Every piece of guidance should trace back to why it exists. If you can't articulate what failure mode a section prevents, it probably doesn't belong.

**Context isolation is structural, not aspirational.** The build composes self-contained skills precisely so consumers don't need progressive disclosure. Source files can reference each other freely. Output files must stand alone.

**Don't add without approval.** New sections, new shared concepts, new checklist items — propose them, don't add them. The owner curates what's in the methodology.

**No content duplication in source.** Shared concepts live in `src/shared/` and get inlined by the build. If the same concept appears in two phase files, extract it to shared.

**Templates and examples are source artifacts.** They get composed into skills or bundled alongside them. Edit in `src/templates/` and `src/examples/`.

## Coherence Checks Before Release

Before tagging a release, verify content coherence:

1. `bun run check` passes (structural validation)
2. Review each `dist/plugin/skills/*/SKILL.md` — does the inlined shared content flow naturally after the phase content?
3. For significant content changes: run a cross-model comparison (e.g. Codex at high reasoning) against previous release output to catch unintended drift
4. Spot-check a `dist/standalone/*.md` file — is it usable when pasted into a chat with no other context?

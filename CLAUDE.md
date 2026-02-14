# Liminal Spec

## Project

Source-based skill with build composition. Edit in `src/`, never in `dist/`.

### Structure

```
src/
  phases/          — Phase-specific content (one file per phase)
  shared/          — Cross-cutting concepts used by multiple phases
  templates/       — Artifact templates (tech design, feature spec)
  examples/        — Verification prompt templates
  commands/        — Plugin command (the /liminal-spec router)
  agents/          — Plugin agents (senior-engineer)
scripts/
  build.ts         — Compose src/ into dist/ (bun run build)
  validate.ts      — Validate dist/ output (bun run validate)
manifest.json      — Maps which shared files each phase skill needs
dist/              — Build output (gitignored)
  plugin/          — Claude Code plugin (skills/ + agents/ + commands/)
  standalone/      — Paste-ready MDs for non-Claude-Code users
```

### Commands

```bash
bun run build       # Compose source into dist/
bun run validate    # Validate dist/ output
bun run check       # Build + validate
bun test            # Run build script tests
```

### How the Build Works

`manifest.json` declares which shared files each phase skill needs. `scripts/build.ts` reads the manifest, concatenates phase content + shared content in declared order, wraps with SKILL.md YAML frontmatter, and outputs to `dist/plugin/skills/<name>/SKILL.md`. It also strips frontmatter and outputs to `dist/standalone/<name>.md` for paste-into-chat distribution.

### Adding/Editing Content

1. Edit the source file in `src/phases/` or `src/shared/`
2. Run `bun run build` to regenerate dist/
3. Test locally: `claude --plugin-dir ./dist/plugin`
4. Run `bun run validate` to verify output

### Release Process

1. Edit source files, commit with conventional commit messages
2. PR → CI validates (build + validate + test)
3. Merge → release-please maintains a release PR
4. Approve release PR → tag created → artifacts published to GitHub Release

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

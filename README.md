# Liminal Spec

A spec-driven development methodology for AI-assisted coding. Designed for features with detailed requirements, complex integrations, or multi-agent coordination.

Liminal Spec runs a phased pipeline where each phase produces an artifact the next phase reads cold — no shared conversation history, no accumulated assumptions. The traceability chain (requirement → test condition → test → code) means when tests go green, you have high confidence the implementation matches the spec.

## When to Use

- New features with multiple components or integration points
- Complex business logic where requirements need precision
- Multi-agent builds where context isolation matters

Not for: quick bug fixes, single-file changes, spikes, or emergency patches. Liminal Spec either runs full or not at all.

## Phases

| Phase | In | Out |
|-------|-----|------|
| 1. Product Research (optional) | Vision/idea | PRD |
| 2. Feature Specification | PRD or requirements | Feature Spec |
| 3. Tech Design | Feature Spec | Tech Design |
| 4. Story Sharding | Spec + Design | Stories + Prompt Packs |
| 5. Execution | Prompts | Verified code |

Most work starts at Phase 2 - if you know what you're building, start there.

Within Phase 5, each story follows: **Skeleton → TDD Red → TDD Green → Gorilla Test → Verify**.

## Key Ideas

**Context isolation.** "Agents" means fresh context with artifact handoff — not roleplay personas. Each phase gets a clean context window. The artifact (document) is the complete handoff.

**Confidence chain.** Every line of code traces back: AC (requirement) → TC (test condition) → Test → Implementation.

**Upstream scrutiny.** The feature spec gets the most review because errors there cascade through every downstream phase.

**Multi-model validation.** Different models catch different things. Artifacts are validated by their downstream consumer and by a different model for diverse perspective.

## Installation

### As a Claude Code Plugin (recommended)

```bash
# Add the marketplace
/plugin marketplace add liminal-ai/liminal-spec

# Install the plugin
/plugin install liminal-spec@liminal-plugins
```

This gives you:

| Command | What it does |
|---------|-------------|
| `/liminal-spec` | Router — presents the phase menu, routes to the right skill |
| `/ls-research` | Phase 1 (optional) — product research and PRD drafting |
| `/ls-epic` | Phase 2 — write a Feature Specification |
| `/ls-tech-design` | Phase 3 — create a Tech Design from a Feature Spec |
| `/ls-story` | Phase 4 — shard into stories and generate prompt packs |
| `/ls-impl` | Phase 5 — execute stories with TDD and verification |

The plugin also includes a **senior-engineer agent** for TDD implementation — rigorous TypeScript development with quality gates (format, lint, typecheck, test).

Start with `/liminal-spec` and it will guide you to the right phase.

### For non-Claude-Code users (BA/PO)

Download standalone files from [GitHub Releases](https://github.com/liminal-ai/liminal-spec/releases). Each file is self-contained and can be pasted directly into Claude Enterprise Chat or any AI assistant.

| File | For | Use when |
|------|-----|----------|
| `product-research-skill.md` | PO, PM, BA | Exploring direction and drafting a PRD |
| `epic-skill.md` | BA, PO | Writing feature specifications |
| `technical-design-skill.md` | Senior Dev, Tech Lead | Creating tech designs from a spec |
| `story-sharding-skill.md` | Tech Lead, Engineers | Breaking features into stories and prompts |
| `implementation-skill.md` | Engineers | Executing stories with TDD |

### From source (for development)

```bash
git clone https://github.com/liminal-ai/liminal-spec.git
cd liminal-spec
bun install
bun run build
claude --plugin-dir ./dist/plugin
```

## Development

```bash
bun install
bun run build       # Compose source into dist/
bun run validate    # Validate output
bun test            # Run tests
bun run verify      # Build + validate + test
bun run check       # Build + validate
```

Edit content in `src/`, never in `dist/`. The build composes phase content with shared references per the manifest and outputs a Claude Code plugin (`dist/plugin/`) and standalone markdown files (`dist/standalone/`). See [CLAUDE.md](CLAUDE.md) for detailed development guidance.

## Project Structure

```
src/
  phases/          — Phase-specific content (one per skill)
  shared/          — Cross-cutting concepts used by multiple phases
  templates/       — Artifact templates
  examples/        — Verification prompt templates
  commands/        — /liminal-spec router command
  agents/          — senior-engineer agent
scripts/
  build.ts         — Compose src/ into dist/
  validate.ts      — Validate build output
manifest.json      — Maps which shared files each phase skill needs
docs/              — Reference material not yet in the build
plugins/           — Committed marketplace-installable plugin directories
```

## Links

- [Releases](https://github.com/liminal-ai/liminal-spec/releases)
- [Changelog](CHANGELOG.md)
- [Development Guide](CLAUDE.md)

## License

MIT

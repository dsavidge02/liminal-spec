# Liminal Spec

A spec-driven development methodology for AI-assisted coding. Designed for features with detailed requirements, complex integrations, or multi-agent coordination.

Liminal Spec runs a phased pipeline where each phase produces an artifact the next phase reads cold — no shared conversation history, no accumulated assumptions. The traceability chain (requirement → test condition → test → code) means when tests go green, you have high confidence the implementation matches the spec.

## Phases

| Phase | In | Out |
|-------|-----|------|
| 2. Feature Specification | Requirements | Feature Spec |
| 3. Tech Design | Feature Spec | Tech Design |
| 4. Story Sharding | Spec + Design | Stories + Prompt Packs |
| 5. Execution | Prompts | Verified code |

Most work starts at Phase 2. Within Phase 5, each story follows: **Skeleton → TDD Red → TDD Green → Gorilla Test → Verify**.

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

Then invoke with `/liminal-spec`.

### For non-Claude-Code users (BA/PO)

Download individual standalone files from [GitHub Releases](https://github.com/liminal-ai/liminal-spec/releases). Each file is self-contained and can be pasted directly into Claude Enterprise Chat or any AI assistant.

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
```

## Development

```bash
bun install
bun run build       # Compose source into dist/
bun run validate    # Validate output
bun test            # Run tests
bun run check       # Build + validate
```

Edit content in `src/`, never in `dist/`. The build composes phase content with shared references per the manifest and outputs a Claude Code plugin (`dist/plugin/`) and standalone markdown files (`dist/standalone/`).

## When to Use

- New features with multiple components or integration points
- Complex business logic where requirements need precision
- Multi-agent builds where context isolation matters

Not for: quick bug fixes, single-file changes, spikes, or emergency patches. Liminal Spec either runs full or not at all.

## License

MIT

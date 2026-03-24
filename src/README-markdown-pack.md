# Liminal Spec — Markdown Pack (v1.0.0)

Paste-into-chat skills for spec-driven software development. Each file is a self-contained skill — paste directly into Claude, ChatGPT, Codex, or any AI assistant that accepts instructions. No installation required.

Liminal Spec guides agents through a structured pipeline: define what you're building (PRD), specify the requirements in detail (Epic), design the architecture (Tech Design), and publish implementable stories. The traceability chain (requirement → test condition → test → code) means when tests go green, the implementation matches the spec.

## Files

| File | Skill | Purpose |
|------|-------|---------|
| `00-prd-skill.md` | ls-prd | Shape product direction into a PRD with epic breakdown and optional Tech Architecture |
| `02-epic-skill.md` | ls-epic | Write a complete, traceable Epic — ACs, TCs, Data Contracts, Story Breakdown |
| `03-technical-design-skill.md` | ls-tech-design | Transform an Epic into an implementable Tech Design |
| `04-publish-epic-skill.md` | ls-publish-epic | Publish an Epic as individual story files with technical notes |
| `06-team-implementation-skill.md` | ls-team-impl | Orchestrate implementation with agent teams and Codex/Copilot CLI |
| `06s-subagent-implementation-skill.md` | ls-subagent-impl | Orchestrate implementation with Claude Code subagents |
| `07-team-spec-skill.md` | ls-team-spec | Orchestrate the full spec pipeline with agent teams |
| `simple-01-story-skill.md` | lss-story | Write a single functional story with epic-quality rigor |
| `simple-02-technical-design-skill.md` | lss-tech | Inline technical design and enrichment for a single story |

## Usage

Pick the file for the phase you need. Paste the entire contents into your AI assistant's system instructions or conversation. Each file is self-contained — no dependencies on other files.

## Changelog

### v1.0.0 (2026-03-24)

Skill pack distribution. Rebuilt orchestration. New upstream pipeline.

**New:** ls-prd (replaces ls-research), ls-subagent-impl
**Rebuilt:** ls-team-spec (procedural learning architect), ls-team-impl (CLI orchestration)
**Refined:** ls-tech-design (multi-doc standard, deviation table), ls-publish-epic (individual story files)
**Removed:** ls-research

See [CHANGELOG.md](https://github.com/liminal-ai/liminal-spec/blob/main/CHANGELOG.md) for full version history.

## Links

- [GitHub](https://github.com/liminal-ai/liminal-spec)
- [Releases](https://github.com/liminal-ai/liminal-spec/releases)

## License

MIT

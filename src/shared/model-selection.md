## Model Selection

Different models excel at different tasks. Use the right model for the job.

| Task | Recommended Model | Why |
|------|-------------------|-----|
| **Orchestration** | Opus 4.6 | Gestalt thinking, manages complexity |
| **Story sharding** | Opus 4.6 | Understands scope, breaks work coherently |
| **Prompt drafting** | Opus 4.6 | Captures intent, writes for other models |
| **Spec/design writing** | Opus 4.6 | Narrative flow, functional-technical weaving |
| **Implementation** | Claude Code / Opus 4.6 | TDD discipline, service mocks |
| **Finicky implementation** | GPT 5x / 5x Codex | Precise, disciplined, less drift |
| **Difficult debugging** | GPT 5x / 5x Codex | Methodical, catches details |
| **Verification** | GPT 5x / 5x Codex | Thorough, catches what builders miss |
| **Code review** | GPT 5x / 5x Codex | Thorough, checks against spec |

### Typical Flow

1. **Opus 4.6** orchestrates, shards stories, drafts prompts
2. **Claude Code subagent** (or Opus with TDD context) executes implementation
3. **GPT 5x** verifies artifacts and reviews code

### Access Methods

- **Opus 4.6:** Claude Code, API, Clawdbot subagents
- **GPT 5x:** Codex CLI (`codex exec`), GitHub Copilot, API
- **GPT 5x Codex:** Codex CLI with `-m gpt-5.2-codex`

→ Reference: `references/prompting-opus-4.6.md`
→ Reference: `references/prompting-gpt-5x.md`

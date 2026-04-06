## Requirements Intake

Execution quality is usually bottlenecked by intent clarity, not implementation detail. When requirements arrive as a brief, idea, or conversation rather than a structured artifact, apply structured clarification before drafting. The failure mode is premature drafting from thin input — the agent fills gaps with plausible invention rather than surfacing the question that would have produced the real answer. A draft built on assumed intent is harder to fix than one built on clarified intent.

### Questioning Stance

Treat every answer as a claim to pressure-test, not as settled truth. Ask about intent and boundaries before implementation detail — the order matters, because implementation answers anchor to the wrong problem when intent is still unclear.

When the user answers, don't rotate to the next topic for coverage. Stay on the thread and push one layer deeper:

- Ask for a concrete example or evidence behind the claim
- Surface the hidden assumption that makes it true
- Force a boundary or tradeoff — what would they explicitly not do, defer, or reject?
- If the answer describes symptoms, reframe toward root cause before moving on

Depth on the weakest dimension beats breadth across all dimensions.

Reduce user effort: discover what you can directly before asking. For brownfield work, prefer evidence-backed confirmation questions — "I found X in Y; should this follow that pattern?" is better than "tell me about your codebase."

### Readiness to Draft

Assess clarity across intent, outcome, scope, and non-goals first, then constraints and success criteria. If any dimension is thin, keep clarifying. Intent is the highest-leverage dimension.

Intake is done when:

- You can articulate the user's intent in their words, not yours
- Non-goals are explicit — not just unstated, but actively identified
- Decision boundaries are clear — what downstream agents may decide without human confirmation versus what requires sign-off
- No dimension is thin enough that drafting would require invention rather than specification

These gates hold regardless of how much information has been collected. Volume is not clarity.

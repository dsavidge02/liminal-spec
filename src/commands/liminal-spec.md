---
name: liminal-spec
description: |
  Spec-driven development for agentic coding. SDLC-phased pipeline with context isolation
  and artifact handoff. Use when building features with detailed requirements, complex
  integrations, or multi-agent coordination.
---

# Liminal Spec

A spec-driven development system for features with detailed requirements and complex integrations. Each phase produces an artifact the next phase reads cold — no accumulated context, no negotiation baggage.

## Phases

| Phase | Skill | Start Here If... |
|-------|-------|-------------------|
| **2. Feature Specification** | `/liminal-spec:epic` | **Most common.** You know what you want to build |
| **3. Tech Design** | `/liminal-spec:tech-design` | You have a complete spec ready for architecture |
| **4. Story Sharding** | `/liminal-spec:story` | Design is done, ready to break into executable work |
| **5. Execution** | `/liminal-spec:impl` | Stories are sharded, ready to implement |

## When to Use

- New features with multiple components or integration points
- Complex business logic where requirements need precision
- Multi-agent builds where context isolation matters

Not for: quick bug fixes, single-file changes, spikes, or emergency patches.

## How It Works

Tell me what you want to build and which phase you're starting from. I'll activate the appropriate skill.

Based on the user's response, invoke the appropriate skill:
- Phase 2 (Feature Specification) → use Skill tool: "liminal-spec:epic"
- Phase 3 (Tech Design) → use Skill tool: "liminal-spec:tech-design"
- Phase 4 (Story Sharding) → use Skill tool: "liminal-spec:story"
- Phase 5 (Execution) → use Skill tool: "liminal-spec:impl"

If the user is unclear about which phase, ask clarifying questions:
- Do they have requirements/a product brief? → Phase 2
- Do they have a complete feature spec? → Phase 3
- Do they have a spec + tech design? → Phase 4
- Do they have stories + prompt packs? → Phase 5

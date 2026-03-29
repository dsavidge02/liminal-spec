# Liminal Spec — Docs Braindump

Raw material for README rewrite, teaching skill, and other documentation. Unstructured. Draw from this.

---

## The Problem SDD Solves

When you ask an AI to build something from a vague brief, it builds confidently — with assumptions it doesn't surface, edge cases it doesn't consider, and behaviors you can't trace back to anything you agreed to. When something breaks, you can't tell whether the code is wrong or the requirement was never defined.

## What SDD Is (One Sentence)

Spec-driven development treats the specification as the primary artifact — code is derived from it, not the other way around.

## What Liminal Spec Is

A skill pack that structures the work so every line of code traces back through a chain — requirement to test condition to test to implementation. When tests pass, you know the implementation matches what was specified, because you can follow the chain from any test back to the requirement it verifies.

---

## Positioning: When SDD Is and Isn't For You

Do you want to build an epic of work that prior to AI coding would take about 5-25 man days to build and you can specify 90-95% of what you need up front with standards, good judgment and mostly understood problems and mostly understood integrations just being put together in a specific desired way? If so, then it may be for you.

Does it need a standard CLI? Does it have relatively standard or normal database patterns? Does it involve standard integration mechanisms and front ends that can be represented on standard desktop apps or standard web app technology? If it falls under this space and you can make some good decisions about who your audience is, what you want them to be able to do and what sort of use cases or user flows would support that and you can reason about all that up front for a few hours — SDD might be a pretty good choice.

Turns out that describes most of enterprise business and a lot of startup founders and a lot of hobbyist businesses.

**Not for you if:**
- You don't know what you want so you want to build small pieces and iterate — not for you
- Quick bug fix — not for you
- Tough novel problems that require interactive experimental iteration — you can use it for the structure but probably can't use it for the hard experimental part. Example: "I want to write a web socket that wraps an agentic CLI and parses the PTY output and turns it into clean readable application events" — the web socket PTY parser probably requires a lot of interactive experimental iteration, even if the surrounding system benefits from SDD

**The judgment call:** Using this framework requires judgment like every other part of building a non-trivial thing that you hope to be of value to yourself and other people. The methodology doesn't remove judgment — it structures where judgment gets applied.

---

## Responding to SDD Criticism

### "It's like waterfall"

Not a criticism. It's a comparison. Waterfall isn't inherently worse than agile unless you lay out broader definitions of good and bad in this space. Agile was a reaction to waterfall — especially bad waterfall. And many things are a reaction to agile — especially bad agile. Saying "it looks like waterfall" is guilt by association — invoking a name the audience already has negative feelings about and letting the connotation do the arguing. It doesn't engage with what SDD actually does.

The legitimate version of that concern: "does the upfront investment in specs pay off relative to the cost, and under what conditions?" That's a real question with a real answer (depends on scope, complexity, team size, and how much correctness matters).

### "Overhead for simple changes"

Not a valid criticism against using a framework for a use case it's not intended for. You don't full-pipeline a one-line bug fix. That's a judgment problem, not an SDD problem.

### "Specs capture intent at a point in time"

Correct. This exposes an invalid assumption that spec documents are supposed to stay authoritative. No documentation stays authoritative without updates. Specs to build an epic are not meant to be authoritative for the current state of an application. It starts as a blueprint for a build. If you choose to update it through the build it becomes a record of what was built at a point in time. It can be useful for reconstructing current state if used correctly.

### "Specs can't capture all context" / "Edge cases emerge at runtime" / "Reality changes faster than specs"

Same pattern: evaluating a build-time artifact against runtime expectations. Like criticizing architectural blueprints because they don't update themselves when you move the furniture. The methodology never claimed specs are the permanent source of truth for the running system.

### Common failure across criticisms

The criticisms cluster around evaluating SDD against assumptions it doesn't make. They impose an expectation — that the spec is the permanent source of truth for the running system — and then criticize the methodology when it doesn't fulfill that expectation. The spec is a blueprint for a build, not a living system description.

---

## The SDD Landscape (March 2026)

### The Arc

Vibe coding (Karpathy, Feb 2025) → plan mode in AI IDEs → spec-driven development as a named practice (mid-2025) → rapid framework proliferation (late 2025 - early 2026) → enterprise adoption starting (2026). Currently in the framework proliferation phase with early enterprise interest.

### Major Players

**GitHub Spec Kit** (~73K stars) — most widely adopted. CLI-driven scaffolding, `constitution.md` as project principles, works across 22+ AI agent platforms. Structured but relatively lightweight — closer to "spec-first" than deep methodology. GitHub's backing gives it distribution.

**BMAD** — most comprehensive in terms of agent roleplay. 21 specialized agents (BA, PM, Architect, Scrum Master, etc.), 50+ guided workflows. "Agent-as-Code" markdown files. Pioneered the multi-agent approach to spec work. Can feel heavy — lots of personas and ceremony.

**Superpowers** (~93K stars) — most opinionated about TDD discipline. Forces agents through structured workflows, 2-5 minute task breakdown, git worktree isolation. Uses persuasion principles to prevent agents from skipping steps. Strong on the implementation side.

**AWS Kiro** — spec-driven IDE. Three-phase workflow (user stories → technical design → tasks). GA late 2025, 250K+ developers in preview. The IDE play — baking SDD into the tool rather than skills/prompts.

### Where Liminal Spec Fits

- Deeper on the spec side than Superpowers (which is stronger on implementation discipline)
- More methodology-driven than Spec Kit (which is more tooling-driven)
- Less persona-heavy than BMAD (which has 21 agent roles vs. Liminal's role-free artifact handoff)
- The confidence chain (AC → TC → Test → Implementation) as a traceable chain is more rigorous than what Spec Kit or Superpowers enforce
- Context isolation via artifact handoff rather than agent personas is a different architectural bet than BMAD's approach

### Enterprise Adoption Landscape

- 67% of teams report extra debugging time during learning phase
- 90-day adoption frameworks with three phases (pilot → expansion → org-wide)
- Microsoft/Accenture studies: 12.92% to 21.83% more PRs per week with structured AI approaches, 96% adoption rate
- Biggest friction: specification writing is unfamiliar to many developers, developer resistance ("AI will replace me"), and the cultural shift from code-first to spec-first

---

## Liminal Spec's Core Concepts

### The Confidence Chain

Every line of code traces back through a chain: AC (requirement) → TC (test condition) → Test → Implementation. Can't write a TC? The AC is too vague. Can't write a test? The TC is too vague. When something breaks, trace from the failing test back to the TC, back to the AC, back to the requirement. This is the differentiator — most SDD frameworks don't enforce traceability at this granularity.

### Context Isolation

Each phase gets a fresh context window. The artifact is the complete handoff — not a summary, not "see above," the full document. This prevents context rot, the "lost in the middle" problem, and negotiation baggage. A fresh agent reads the artifact as-is with no conversation history. The artifact IS the interface between phases.

### Upstream Scrutiny

The epic gets the most review because errors cascade through every downstream phase. Fix problems at the spec level — cheapest to fix there, most expensive in code. The scrutiny gradient: epic (every line) → tech design (detailed review) → stories (key things + shape) → implementation (spot checks + tests).

### The Pipeline as System Design

PRD and arch are the human's front-loaded investment — the place where you put in the most thinking so the pipeline can run with progressively less re-engagement. The epic is the pivot — the last functional artifact, where requirements become testable. Tech design is where functional meets implementation — the model reads the epic and designs how to build it. Stories are the delivery units — each one self-contained enough to implement from. Each phase produces an artifact the next phase reads cold — that's context isolation, and it's what keeps quality consistent across long builds.

PRD and arch are peers, not sequential. The epic is the pivot. Tech design is dual-role (validator + designer). The team skills orchestrate this with agent teams.

### Dimensional Reasoning (v1.1.0)

Higher-order reasoning prompts at key structural decisions. When a model chooses an organizing structure — top-tier surfaces, module boundaries, feature boundaries, story partitions — it tends to lock onto the first compelling axis and settle the decision before weighing competing considerations. The check prompts the model to enumerate the tensions for the specific system, identify where they pull toward different answers, and weight which should win before committing. It names the process, not the factors — those are different every time.

---

## Target Audiences

### The vibe coder who's hit the wall
Built something with AI, it works-ish, but they can't verify correctness or trace behavior back to intent. They've heard of SDD and want to try it.

### The SDD practitioner shopping frameworks
Knows the space, has maybe tried Spec Kit or BMAD, wants to understand what Liminal Spec does differently.

### The enterprise team evaluating approaches
Needs to understand rigor, traceability, team workflow, and how this fits into existing Agile/Scrum processes.

### The existing user
Needs reference material, pipeline tables, installation instructions.

---

## README Structure (Proposed)

1. What problem this solves (2 paragraphs)
2. Core concepts — confidence chain, context isolation, upstream scrutiny
3. How the pipeline works (narrative, then table for reference)
4. Where to start (entry-point routing based on what you have)
5. Running the pipeline — manual (individual skills) vs. orchestrated (team skills)
6. Skills reference table
7. Installation
8. Development
9. Links

Lead with the problem and concepts, not the skill catalog. Teach before you reference.

---

## Teaching Skill Ideas

An `ls-guide` or `ls-start` skill. Interactive. When someone loads it, it asks what they're trying to build, what artifacts they have, what their experience level is. Recommends which skills to use, explains relevant concepts, walks through the first step. Knows enough about all 8 skills to route correctly without duplicating each one.

Previously existed as a command prompt in the plugin era, removed when the plugin marketplace was dropped. A skill is the right container — no marketplace needed, just a SKILL.md that knows the catalog and helps you navigate it.

---

## Writing Style Notes

### For the README
- Lead with the problem, not the solution
- Situate in the SDD landscape without being a comparison page
- Name SDD as the category (don't describe OO problems without saying OO)
- Don't pitch against competitors — describe the approach and let readers compare
- The confidence chain and context isolation are the two differentiating ideas

### General
- The methodology doesn't remove judgment — it structures where judgment gets applied
- Don't oversell. The honest positioning (5-25 day epics, 90-95% specifiable upfront, standard patterns) is more credible than "works for everything"
- Acknowledge what it's not for — quick fixes, experimental iteration, novel algorithm work

---

## Origin Story and Author Context

Lee Moore. Lead Engineer (Staff → Principal) at Travelers, Fortune 100, largest business insurance provider. Nearly 30 years senior engineer and tech lead experience in business apps. Fortune 500 most of career.

**The path to Liminal Spec:**
- Stopped writing code in 2020, stayed very technical — designed patterns, ran teams that made patterns
- May 2025: turned general AI obsession into AI coding. Hadn't coded in 4-5 years so had no friction going full agentic with minimal code reading. No muscle memory fighting the new paradigm.
- June 2025: dev team selected for SDLC+ primarily because Lee was loud and engaged on AI
- Gained enterprise prominence helping people and displaying AI expertise. Productivity started to multiply — far more productive as employee and coder, amplified by the nearly 30 years of senior engineer experience
- Started to realize the agentic work needed a process. Modeled it on agile practices at work. This became the spec-driven approach.
- Got good enough that for the first time on the team (2 years in), started doing board work. First PI using own spec-driven process with TDD: 10 points in the first iteration while doing all normal tech lead responsibilities.
- Now probably top 2-3 AI engineer at Travelers, top 1-2 Claude Code user. 350-person session list, typically 100-140 people show up for 2-hour Friday afternoon AI sessions.

**Current adoption:**
- Not known outside Travelers yet
- Inside Travelers: getting exposure on Lee's team and another staff engineer's team
- The specs grew from 1 skill with many progressive disclosure files → parsed into separate skills for standalone use in claude.ai

**The enterprise distribution constraint:**
- POs and BAs don't have Claude Code but they do have Claude Enterprise
- Claude Enterprise doesn't have skills enabled
- So the markdown pack exists: every skill is also distributed as a single standalone markdown file
- BAs and POs use ls-epic and ls-publish-epic as standalone prompts they drag into a claude.ai project or insert into the chat
- This isn't a fallback — it's a first-class distribution channel for non-CLI users in the enterprise

**What this means for the audience:**
- The methodology was born from enterprise agile practice, not indie dev culture
- It was designed by someone who stopped writing code before AI coding existed and came back with no legacy coding habits to unlearn
- It serves people who already think in terms of stories, epics, acceptance criteria, and PIs — the agile vocabulary is native, not borrowed
- The enterprise audience isn't an afterthought — it's the origin. The indie/startup/hobbyist audience is the expansion.

---

## Distribution Model

### Two Packs, Two Audiences

**Skill Pack** (`liminal-spec-skill-pack.zip`) — installable skill directories for Claude Code, Codex, and other skill-aware AI CLIs. Unzip into `~/.claude/skills/` or equivalent. Each skill is a directory with a `SKILL.md`. The CLI discovers and triggers skills based on descriptions in the YAML frontmatter.

**Markdown Pack** (`liminal-spec-markdown-pack.zip`) — per-skill standalone markdown files. No installation. Paste into any AI assistant that accepts instructions: Claude.ai projects, ChatGPT, Codex, Gemini. Every skill works as a standalone prompt — all shared concepts are inlined by the build so no progressive disclosure or reference file loading is needed.

### Why Both Exist

The markdown pack isn't a degraded version of the skill pack. It serves users who don't have CLI access — POs dragging ls-epic into a Claude.ai project, BAs using ls-publish-epic to shard stories, enterprise users on Claude Enterprise without skills enabled. The build system composes both from the same source files. Same content, different delivery mechanism.

---

## Audience Segments (Expanded)

### 1. Enterprise agile teams adopting AI coding
The origin audience. Think in epics, stories, ACs. Already have POs, BAs, tech leads. Need a structured approach to AI-assisted development that maps to their existing vocabulary and ceremonies. The confidence chain (AC → TC → Test → Implementation) translates directly to how they already think about acceptance criteria and definition of done. Key concern: traceability, governance, and fitting into existing SDLC processes.

### 2. Tech leads / staff engineers going agentic
Senior engineers who've been designing systems and leading teams, now incorporating AI coding into their workflow. They have the domain knowledge and architectural judgment — they need a framework that leverages that experience rather than trying to teach them engineering from scratch. Liminal Spec's emphasis on judgment at the right altitude speaks directly to this audience.

### 3. The vibe coder who's hit the wall
Built something with AI, it works-ish, but can't verify correctness or trace behavior back to intent. Heard of SDD. Needs to understand what structured development gives them without being talked down to.

### 4. SDD practitioners shopping frameworks
Know the space. Have tried Spec Kit, BMAD, Superpowers. Want to understand what Liminal Spec does differently. Evaluating the confidence chain, context isolation, and the enterprise-origin design philosophy.

### 5. POs and BAs (non-CLI users)
Use the markdown pack in Claude.ai or Claude Enterprise. Their entry points are ls-epic (writing or reviewing functional specs) and ls-publish-epic (sharding stories). They don't need to understand the full pipeline — they need their specific skills to work as standalone prompts.

### Primary dial-in
Enterprise agile teams and senior engineers going agentic. These are the people who already have the judgment and vocabulary — they need the framework to structure where that judgment gets applied. The indie/hobbyist audience benefits from the same tools but the language and assumptions should be calibrated for people who know what an acceptance criterion is.

---

## How the Methodology Actually Evolved

The process wasn't grounded in agile initially. The agile vocabulary came after the structure emerged.

Working agentic, things naturally broke down into larger chunks that slowly became smaller chunks. The chunks needed names. Over time it became clear that epic, stories, tech designs, PRDs, and tech docs fit at the right levels. But the definitions aren't classic agile definitions — they're shaped by how the artifacts need to function together in the pipeline, not by textbook definitions.

**Test conditions came from testing practice, not from spec methodology.** On a prior rating domain, when setting up heavy testing, Lee pushed testers to create test condition documents outside of the testing framework — focused on what was functionally being tested, not how. From those test conditions, testers should be getting business buy-in. Over time this became: associate test conditions with acceptance criteria in the epic. Line-level ACs with full TC permutations create the traceability chain.

**Publish-epic was born from a readability problem.** All the line-level acceptance criteria and all the detail and all the permutations of test conditions laid out cause extremely long, verbose, hard-to-digest epics for POs and BAs. So publish-epic was created to break it down — individual story files with Jira section markers. The PO/BA-friendly format wasn't designed upfront; it was a reaction to a real consumption problem.

**The skills were initially one skill with progressive disclosure files.** Parsed into separate skills because POs and BAs needed standalone prompts for claude.ai — they can't use Claude Code skills.

---

## The Key Power Player

The person who makes this work best is the **senior engineer or tech lead who also has sufficient product skills to understand functional versus technical**. This is the critical user profile.

Why this person exists more often than you'd think: engineers have to turn functional and business requirements into code. To do that well, you have to understand the functional and business side. If you don't, the business will tell you why you're wrong. You're much more likely to find engineers who can adapt to the product side than product owners who can adapt to the engineer side.

When you have deep technical skills AND product understanding in one person, that person becomes ridiculously valuable with SDD. They can write the PRD, write the epic, write the tech design, and implement — all with the same judgment applied at every altitude. Brandon (staff engineer, Deluxe Property Build at Travelers) picked up the approach and became powerful with it overnight. To him, within the enterprise context, it was clearly the superior way to go.

**The PO/BA reality:** For many teams, unless the PO is exceptional and creative, they're finding themselves as a third wheel on a lot of this. The skills ARE friendlier for POs than raw spec work — ls-epic forces so many clarifying questions and has such specific criteria that even a PO who's new to it can produce good output. They can use ls-epic to create epics and ls-publish-epic to shard stories. It's not that hard. But the depth of thinking through implications and details — that tends to be a senior engineer strength.

The industry SDD conversation doesn't address this at all. BMAD, Spec Kit, Superpowers — they assume the developer is the only user. The cross-functional team dynamic (PO reviews, BA shards, engineer implements) isn't part of their model.

**Two collaboration patterns emerging:**
1. Senior engineer writes epics and tech designs, POs and BAs review and shard stories from them. This is the current dominant pattern — it's faster and produces better output, but it concentrates the spec work.
2. Non-technical PO and tech lead collaborate on epics together, each contributing their domain. This works but requires the PO to engage with more detail than traditional agile asks of them.

---

## Field Validation

### What people actually say

The epics generated with this process consistently get called out in retros by developers as very effective — far more detail than people typically expected, and the test conditions specifically make builds easy. Developers say the specs make it clear what to build and how to verify it.

### Adoption pattern at Travelers

- Lee's team: generating epics and tech designs for the AI enablement team. POs and BAs took the specs because the team was new and they didn't have epics ready. Quality was immediately recognized.
- Brandon's teams (Deluxe Property Build): pushing 3-4 integration teams to adopt. Getting POs on board — at least one team's PO has been happy with what the process creates.
- Broader: in the value stream (~1/3 of Business Insurance), teams moving fastest are either using Liminal Spec or using approaches with similar principles. The pattern is converging independently.

### Training observation

It's easier to train senior-ish engineers on the methodology than to train POs/BAs on it. Engineers already think in terms of edge cases, test conditions, and system boundaries. POs think in terms of user value and business outcomes. The translation from business outcome to testable specification is the skill gap — and it's a gap that runs in the engineer-to-product direction more naturally than the reverse.

---

## The Velocity Story (PI2, March 2026)

### The numbers

Team formed with high velocity from the start. Trajectory over PI1:
- Iteration 1: 28 points (Lee going hard, 4 engineers)
- Middle iterations: Lee stepped back from points, focused on getting people running
- Iteration ~4: 33 points (Lee doing minimal, had people out)
- Committing to 38 points in current iteration

Then the level-up: 18 points completed in the first two days of the iteration. Expect another 22 points by end of day Wednesday — over commitment by halfway through the iteration. The team is now tackling work by the epic, not by individual stories.

### The 18-point proof

One epic, ~18 points, 5 stories. Senior engineer (Jude) wrote his own epic and tech design. Lee ran publish-epic. Then challenged Jude: "Don't build this one story at a time. Run ls-team-impl-cc."

Jude used agent teams. All 5 stories across 18 points — initial automated build and verification completed in 3.5-4 hours with Jude only occasionally approving things. Next day: fine-tuning, pushing to PR. Lee used agent swarms to fully validate the entire feature on the PR. Back and forth a couple of times. Merged on Friday.

This is the proof point. Not a demo, not a prototype — real board work, real PI, real team, merged to the codebase.

### The training arc

**January:** Claude Code rolled out broadly at Travelers. Lee did 4-5 training sessions over 3 weeks.

**Early February:** Final session was on spec-driven development. Showed actual epics and tech designs, the conversations used to create them, and the full product buildout. This was the session that scared people. A senior engineer asked: "Are you saying we're going to come to a point where we don't even look at the code?" Lee's answer: "People on the cutting edge, that's not even a controversial question. In six months this will be a lot more obvious."

The session was less interactive than usual. People who keep coming and know more weren't that scared. But the general impression: showed too much too fast, some muggles got scared. Lee put head down and kept building.

**Now (late March):** Multiple teams are coming to Lee asking "now that we're into planning, how do we use all of this stuff with our epics and our stories for execution?" Teams are trying to book time. The demand shifted from curiosity to "show me how to actually do this."

### Friday training (upcoming)

This is the next major training moment. Lee will show what the team has been doing — the 18-point epic, the team orchestration, the velocity numbers. The documentation being built now needs to serve these people too.

**The audience for this training:**
- Technical engineers getting started (primary)
- Courageous POs trying to get something moving on their team (can use ls-epic, ls-publish-epic to start contributing)
- Teams who've been watching and are now ready to try

---

## The Real Landscape for Documentation

The documentation needs to serve a broad landscape simultaneously:

1. **Internal Travelers engineers** at Friday training who are seeing this for the first time or the second time and are now ready to try
2. **Engineers at other companies** finding this through GitHub
3. **SDD practitioners** evaluating frameworks
4. **POs and BAs** who are courageous enough to try the functional-side skills
5. **Senior engineers / tech leads** who recognize themselves as the key power player

Can't comprehensively cover all of these. But the documentation should present a landscape — where these things are, what this is, who it's for — and let people self-select into the depth they need.

The teaching skill (ls-guide) could do the interactive routing that the README can't. The README presents the landscape. The teaching skill meets you where you are and walks you in.

---

## What's Timely

The timing matters. This isn't "we built a framework and here's the docs." This is:
- Teams are asking NOW how to use this for their current PI planning
- The methodology just proved itself with real velocity numbers that the team witnessed
- The next training is Friday
- The demand shifted from "interesting, tell me more" to "show me how to do this"

The documentation is arriving at exactly the moment people are ready to use it. That changes the tone — it's not evangelism, it's enablement.

---

## Documentation Consumer Test

Same principle as the methodology's consumer test for specs: each doc artifact has a reader, a capability gap, and a measurable outcome. If you can't say "after reading this, the reader can do X that they couldn't before," the doc doesn't have a job.

### README
- **Reader:** Someone who found this on GitHub, got a link from a colleague, or heard about it at a Friday training
- **Before:** "What is this? Is it for me?"
- **After:** Understands what SDD is, what Liminal Spec's approach is, whether it fits their situation, and where to start
- **Capability gap closed:** "What is this" → "I know whether and how to try it"
- **Not trying to do:** Teach them the methodology. That's the skills' job.

### Teaching Skill (ls-guide)
- **Reader:** Someone who installed the skills or has the markdown pack, sitting in front of a project
- **Before:** "I have these tools but I don't know which one to use or how to start"
- **After:** Knows which skill to load for their situation and role, has started their first artifact
- **Capability gap closed:** "I have the tools" → "I'm using the right tool for my situation"
- **Routes by:** Role (tech lead, developer, PO, BA), what artifacts exist already, ambition level ("I want to review specs" vs "I want to write specs")

### Individual Skills (ls-epic, ls-tech-design, etc.)
- **Reader:** Someone actively producing an artifact
- **Before:** "I need to write an epic / tech design / etc."
- **After:** Has a complete artifact that meets the methodology's quality bar
- **Capability gap closed:** "I need to produce this" → "I have a complete, validated artifact"
- **These already exist and work well.** The gap is getting people TO the right skill with the right context, not the skills themselves.

### Friday Training Materials (not a permanent doc, but a doc artifact)
- **Reader:** Travelers engineers at various comfort levels, some seeing this for the second or third time, now ready to try
- **Before:** "I've heard about this, I'm curious or skeptical, I don't know how to start on my team"
- **After:** Understands the approach well enough to try it on their next piece of work, knows where to get the skills, knows who to ask
- **Capability gap closed:** "Interested observer" → "Ready to try it this PI"
- **Key constraint:** Can't scare the muggles. Show the approach and let capable people see the implications. Don't front-load the conclusion.

### The principle
Who is your reader. What do they need. What will they be able to do after reading your documentation that they couldn't before. And why is that a good thing.

The "why is that good" is critical. It either needs to be self-evident or clearly articulated. "You can write an epic with full AC/TC traceability" is a capability. Why is that good? Because when your agent builds from that epic, every test traces to a requirement you approved — so when tests pass, you know the build matches what you specified. Without that, you're trusting the agent's interpretation and hoping.

If you skip the why, you're assuming the reader already values the capability. For the engineer who's still figuring out why they'd write specs instead of just prompting the agent, that assumption fails. The why connects the capability to the reader's actual problem.

This is the same discipline the methodology applies to specs, applied to the documentation itself.

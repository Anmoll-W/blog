---
date: 2026-04-16
tags: [vault, ai, systems, agents, skills]
series: vault-as-os
---
<!-- source_session: 2026-04-16_claude-skills-agent-wiring -->

# My AI Agents Had Identity. They Needed Methodology.

I had built 13 AI personas. Each one had a name, a domain, a set of constraints, a clear "never do" list. They knew who they were. What they did not have was structured methodology for the hard parts.

Ask the QA agent to run a test plan and it would write test cases from scratch, forward-looking, optimistic — the same way most junior testers do. It would never start by assuming the feature had already failed. Ask the code review agent to flag problems and it would check the things it remembered to check. Security, Supabase migration safety, TypeScript correctness. It had no systematic security scanner wired in. Ask the eval agent to challenge a decision and it would push back — but from its own instincts, not from five distinct angles the way a structured red-team forces you to do.

Identity without methodology produces inconsistent results at exactly the moments that matter most.

---

## The Trigger

A LinkedIn carousel by [@okaashish](https://www.instagram.com/okaashish/) surfaced six Claude skills described as tools for product teams. The paths pointed to a public GitHub repository — [jeffallan/claude-skills](https://github.com/jeffallan/claude-skills) — with 65+ skills organized by domain: workflow, API architecture, frontend, testing, DevOps, security.

I had known about Claude skills as a concept. I had not done a systematic pass on what was publicly available and what was actually worth installing.

This was the moment to do that.

---

## What a Claude Skill Actually Does

A skill is a Markdown file with a structured identity: a name, a description, trigger phrases, and a workflow. When you invoke it, it loads into the conversation as a specialized operating mode. The model follows its methodology rather than improvising.

The difference between an agent that "does security checks" from memory and an agent with a security skill wired in is the difference between a developer who remembers to check for SQL injection most of the time and one who runs a checklist that covers OWASP Top 10, secrets exposure, auth gaps, and XSS on every review, every time.

Skills are installable. Reusable. Composable. And there is a growing public ecosystem of them.

**Where to find skills:**
- [jeffallan/claude-skills](https://github.com/jeffallan/claude-skills) — 65+ full-stack development and workflow skills with structured SKILL.md files
- [BehiSecc/awesome-claude-skills](https://github.com/BehiSecc/awesome-claude-skills) — curated directory of community-built skills across dozens of categories
- [Smithery.ai](https://smithery.ai) — skill marketplace with 128,000+ indexed skills, searchable by category

---

## The Six Skills I Installed

All six come from [jeffallan/claude-skills](https://github.com/jeffallan/claude-skills), MIT license.

**[Feature Forge](https://github.com/jeffallan/claude-skills/tree/main/skills/feature-forge)** — runs a structured requirements workshop from two angles: PM perspective (user value, success metrics) and dev perspective (technical feasibility, edge cases). Produces EARS-format functional requirements, acceptance criteria, and an implementation checklist. The alternative is a spec written from one person's memory of what good specs look like.

**[Spec Miner](https://github.com/jeffallan/claude-skills/tree/main/skills/spec-miner)** — reverse-engineers specifications from an existing codebase. Traces data flows, maps API surfaces, and documents the implicit requirements hiding in the implementation. Useful when you inherit code with no docs, or when you need to understand what a system actually does before changing it.

**[The Fool](https://github.com/jeffallan/claude-skills/tree/main/skills/the-fool)** — structured critical reasoning across five modes: Socratic (expose assumptions), Falsification (test the evidence), Dialectic (build the strongest counter-argument), Pre-mortem (find failure modes before they happen), and Red team (adversarial attack from outside). Named after the court jester — the one who could speak truth to the king.

**[Architecture Designer](https://github.com/jeffallan/claude-skills/tree/main/skills/architecture-designer)** — thinks like a principal architect. Evaluates trade-offs, writes Architecture Decision Records, produces system diagrams. Turns "help me choose between these two approaches" into a documented decision with context that future-you can read.

**[API Designer](https://github.com/jeffallan/claude-skills/tree/main/skills/api-designer)** — REST and GraphQL API design with OpenAPI 3.1 specifications. Covers resource modeling, versioning strategy, pagination patterns, error handling. Produces a contract, not just a sketch.

**[Microservices Architect](https://github.com/jeffallan/claude-skills/tree/main/skills/microservices-architect)** — distributed systems design using Domain-Driven Design, bounded contexts, saga patterns, event sourcing, CQRS. Includes validation checkpoints at each stage: service boundaries, data ownership, resilience patterns, observability.

---

## The Wiring Decision

Installing the skills was the easy part. The question was how to wire them.

There are two ways an agent can use a skill:

**Auto-invoked** — always, at a specific step, without being asked. No condition required. The skill is part of the agent's standard operating procedure.

**Situational** — invoked when a named condition is met. It is available and the agent knows to reach for it, but it does not run by default.

Getting this distinction wrong in either direction creates problems. An auto-invoked skill on every code review is fine. An auto-invoked architecture review on every commit is noise. A security scan that only runs "when it seems relevant" is a gap.

The framework I used: if the check is a quality gate that should never be skipped, it is auto-invoked. If it adds value only in specific contexts, it is situational.

---

## How Each Agent Changed

**Code review** now auto-invokes two skills on every review:

1. `security-review` — a dedicated security pass covering OWASP Top 10, SQL injection, XSS, secrets exposure, and auth gaps. Appended as a separate `### Security Pass` section in the output.
2. `simplify` — checks for over-engineered code, unnecessary abstractions, and dead complexity after the main review. Appended as `### Simplification Opportunities`.

Situationally, it invokes `the-fool` when the diff includes an architectural decision, and `spec-miner` when reviewing code in an undocumented area.

**QA check** now auto-invokes one skill before writing any test cases:

`the-fool` in Pre-mortem mode — "Assume this feature has already failed in production. What went wrong?" The output seeds the edge case and error state checklist. The shift is subtle but significant: forward-looking test design catches what the spec describes. Pre-mortem thinking catches what the spec forgot to describe.

Situationally, when no acceptance criteria are provided, `feature-forge` derives EARS-format requirements first rather than letting the agent invent criteria from thin air.

**Draft post** now auto-invokes two skills on every post:

1. `marketing-skills:copywriting` — structural copywriting frameworks applied before writing the hook and body.
2. `marketing-skills:copy-editing` — final pass that catches AI writing patterns (there is a list of 21 categories), passive voice, and filler. This runs before output.

---

## What Changed Across the Persona Layer

Beyond the three stateless agents, I also audited all 13 personas for skill gaps.

The pattern was consistent: personas had strong identity and domain knowledge but were under-equipped for cross-cutting concerns. Security review belonged everywhere that code was being touched. Structured critical reasoning belonged everywhere that decisions were being made. Copywriting frameworks belonged everywhere that copy was being produced.

`the-fool` ended up wired to nine different personas and agents — the eval agent, QA, design, marketing, the solo business architect, the team operations agent, the AI architect, and both stateless agents. Not because every session uses it, but because every session where a decision is being made should have it available, and every agent that runs a mandatory check should have it auto-invoked at that step.

The six personas that had no borrowable skills section got one. Rex (solo business architect) got the most additions: `grill-me` for pressure-testing business ideas, `the-fool` for stress-testing models, and a stack of marketing skills for the sales copy and outreach work that defines the first 90 days of a solo business.

---

## The Thing I Had Not Checked

When I looked at the QA agent before this session, it was doing what it had always done: writing test cases from scratch based on what it knew. It had no instruction to start by imagining failure. The feature-under-test was always implicitly assumed to be correct until proven otherwise.

That is the wrong starting assumption for a QA agent.

The Pre-mortem mode in `the-fool` inverts this: the feature has failed, what went wrong? It is a different cognitive posture and it surfaces a different class of bugs — the ones that live in the gap between what the spec described and what production actually does.

I had built a QA agent and never wired in the thing that makes pre-mortem thinking systematic. It was not obvious until I had a skill that named the gap.

---

## What I Learned

**Technical:** Claude skills from public GitHub repositories are installable in minutes. The harder work is the wiring decision — not what skills exist, but which agent needs them at which step, and whether that step should be mandatory or conditional.

**Decision:** Auto-invoke vs. situational is the right frame for any capability you add to an agent. The question is not "could this agent use this skill?" but "should this agent always run this skill at this step, or only when a named condition is met?" The first question leads to a long borrowable-skills list that never gets used. The second question leads to a short auto-invoke list that always runs.

---

## Related

- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the foundation this extends: how 11 personas replaced isolated agents and what the coordination architecture looks like
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 persona redesign that this session builds on top of
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — Vera, the eval persona, is one of the agents extended in this session; her `the-fool` integration is the most direct application
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer that runs these agents on schedule
- [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](second-automation-layer.md) — the latest automation expansion, which added quality gates that now benefit from the skill wiring here
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — what came next: a performance audit that found five agents not executing their protocols, and how self-correcting behaviors were baked into each identity

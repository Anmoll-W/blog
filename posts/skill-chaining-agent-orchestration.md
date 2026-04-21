<!-- source_session: 2026-04-19_skill-chaining-agent-orchestration -->

# The Agents Were Ready. The Coordination Was Not.

*2026-04-19 · claude, obsidian, agents, orchestration, vault-os · Vault as OS*

I had 13 agents. A skills registry. A routing table in CLAUDE.md that matched signal words to personas. Every piece was built, labeled, and documented.

And every session, I still had to tell each agent what to do manually.

"Run copywriting on this." Then: "Now run copy-editing." Then: "QA this draft." Three separate commands for work that should have chained automatically. The system looked coordinated on paper. In practice, I was the coordinator.

The missing layer was not more agents. It was an orchestration contract — a way for skills to chain, agents to hand off, and automation to run without me in the loop.

This is how I built it.

---

## The Problem with Islands

Each of my agents was well-defined. Draft-post knew how to write. Code-review knew how to critique. QA-check knew how to stress-test. But none of them knew the others existed.

When I finished a draft, I manually invoked QA. When QA flagged issues, I manually routed them to code-review. When automation ran overnight, there was no way to know what had failed until I opened the vault the next morning and noticed the brief was missing.

Islands. Thirteen of them. Each one capable. None of them connected.

The question was not "do we need coordination?" The question was "where should the coordination intelligence live?"

---

## Three Approaches, One Right Answer

I considered three architectures:

**Declarative chaining** — each skill declares `chains-to:` in its frontmatter. Skills wire themselves. Elegant in theory, fragile in practice. Static frontmatter cannot handle conditional routing. If a task is a post, chain to Maya. If it is code, chain to Quinn. Frontmatter cannot make that call.

**Signal-based routing** — skills emit structured output tags, a dispatcher reads them and routes. Loosely coupled. But this requires two things to stay in sync: every skill must emit a consistent signal format, and the dispatcher must parse it correctly. Two drift surfaces instead of one.

**Central orchestrator** — one skill reads intent, queries a registry of available skills, builds an execution graph, and dispatches. All routing intelligence in one place. Adding a new skill means updating the registry once. The orchestrator picks it up automatically.

The pre-launch-audit skill had already proven this model. It dispatches six specialists in parallel, collects their findings, then routes to Vera for verification. That is an orchestrator. I needed to generalize the pattern, not invent something new.

Central orchestrator won. Not because it is the most elegant — declarative chains are more elegant. It won because it handles conditionals, can be updated in one place, and fails loudly when something breaks.

---

## The Architecture

Three components:

**The registry** — `Knowledge/skills-registry.md`. Every skill and agent in the system, with a consistent schema: capabilities, requires, produces, conditions, priority, parallel-safe flag, and a `last-verified` date. The registry is the only place where routing intelligence lives. If a skill is not in the registry, the orchestrators do not know it exists.

**The live orchestrator** — `Skills/orchestrator-live.md`. Reads user intent, queries the registry, builds a graph of which skills to run in what order, and executes it. Split into two strict phases: PLAN (builds the graph, states it in two lines, proceeds) and EXECUTE (runs tier by tier, collects outputs, merges results). The phases never blend. Blending them is what causes orchestrator prompts to collapse into inconsistent behavior as the system grows.

**The auto orchestrator** — `Skills/orchestrator-auto.md`. Called by LaunchAgent scripts. No human in the loop. Handles lock files (prevents double-runs if a previous run is still active), vault mount checks, retry logic per skill, and health logging. After every run, it writes a structured entry to `Knowledge/automation-health.md`. The next morning briefing reads that file first and surfaces any failures.

---

## What Vera Caught

Before writing a line, I ran a structured pre-mortem. Five failure modes came back.

**Registry drift.** The registry is a markdown file. Skills evolve. Nobody updates the registry when they update a skill. Within weeks, the orchestrators are routing based on stale entries. Fix: every entry has a `last-verified` date. Weekly-review now validates freshness as its final step. Entries older than 14 days get flagged.

**Orchestrator prompt collapse.** Seven responsibilities in one skill file — reading intent, querying registry, detecting cycles, planning tiers, executing, validating handoffs, merging outputs. As skill count grows from 20 to 50, the prompt grows and Claude follows it inconsistently. Fix: strict PLAN/EXECUTE phase split. Each phase is small enough to hold in context at once.

**Silent automation failures.** When an overnight run fails, there is no alert. You find out when you notice the morning brief is missing. Fix: the auto-orchestrator writes a health log entry after every run. Morning-briefing reads it first and surfaces any failures at the top of the brief.

**Handoff contract fragility.** Agent coordination depends on every agent consistently outputting a `## Handoff` block. One agent update that drops the section breaks chaining silently. Fix: the orchestrator validates Handoff presence before marking any agent complete. Missing Handoff triggers a `HANDOFF_MISSING` log.

**Query ambiguity.** Multiple skills match the same intent. Which wins? Without a rule, the orchestrator guesses differently each time. Fix: every registry entry has a `priority` field and a `conditions` block. Disambiguation is deterministic: most matching conditions wins, ties broken by priority.

Each fix was designed into the spec before a single file was written.

---

## The Handoff Contract

Every agent in `~/.claude/agents/` now ends with this:

```
## Handoff
status: complete | needs-review | blocked
produces: [outputs]
next-agent: [agent-id or empty]
context: |
  One line of what the next agent needs to know.
```

The orchestrator reads this after every agent run. If `next-agent` is set and valid, it dispatches. If `status: blocked`, it surfaces immediately. If the Handoff section is missing entirely, it logs and continues — but you know it happened.

This is the coordination contract. Not elegant prose in a system prompt. A structured output field that the orchestrator validates mechanically.

---

## What Changed

Before: 13 agents, manual coordination, no visibility into overnight automation.

After: say "write a post and QA the draft" in a session and the live orchestrator builds the chain — draft-post → qa-check — states it in two lines, and executes. Say nothing, and the automation runs overnight, chains its own skills, and surfaces any failures in the next morning brief.

The agents did not get smarter. The coordination layer got built.

---

## What I Learned

**Technical:** An orchestrator that blends planning and execution becomes unreliable as complexity grows. Strict phase separation — PLAN builds the graph, EXECUTE runs it — keeps each phase small enough to follow consistently. This is not a preference; it is the difference between a system that works at 20 skills and one that works at 50.

**Decision-making:** The right architecture is usually the one that already works somewhere in your system. Pre-launch-audit was already an orchestrator. I generalized a proven pattern instead of designing a new one. When you find yourself inventing elegant abstractions, check whether a simpler version of the thing already exists in your codebase. It usually does.

---

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the LaunchAgent foundation that the auto-orchestrator now runs on top of
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — how skills were wired to agents before the orchestration layer existed
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — Vera's role in the pre-mortem that shaped this design
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — the self-correction layer that runs inside the chains this orchestrator coordinates
- [The Vault Was Organized. The Files Were Not.](vault-structural-drift.md) — the session immediately after this one: a nightly structural hygiene automation built using the same skill-file pattern the orchestrator relies on

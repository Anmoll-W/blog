---
title: "Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas"
date: 2026-04-10
tags: [ai-agents, personas, prompt-engineering, vault, obsidian, knowledge-management]
series: vault-as-os
series_order: 2
source_session: 2026-04-10_agent-team-v2-launch
related:
  - three-claude-tools-one-vault.md
  - vault-os-that-runs-itself.md
  - how-i-retired-notion.md
  - persona-layer-architecture.md
  - parallel-agents-ai-stack.md
  - formula-hooks-kill-metrics.md
---

# Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas

> The problem was not that the agents lacked capability. The problem was that they did not know what each other had learned.

## Context

By early April 2026, my Obsidian vault had seven AI agents: an engineer, a product manager for LinkWhisper, a marketing director, a finance expert, a business analyst, a travel planner, and a vault architect. Each one had an identity file and a memory file. Each one had accumulated real knowledge from real sessions.

The problem was isolation. Alex (the engineer) had learned that Supabase returns numeric columns as strings and you need to parse them explicitly before doing arithmetic. Maya (the marketing director) had learned that overusing a single hook type causes engagement to collapse even when the data said it was the top performer. These lessons were real and hard-won.

But Zara (the finance expert) started fresh every time. Sam (the business analyst) started fresh every time. When a project touched multiple domains, the relevant agent had to be manually loaded by name. There was no shared understanding. There was no memory that crossed agent boundaries.

This was the core architectural problem: agents learning in isolation.

## The New Architecture

I designed and built a complete agent team system in a single session. The changes were:

**Four new agents added:** Quinn (QA engineer), Nova (build project manager), Ren (creative designer), Harper (HR and team health).

**Seven existing agents upgraded:** all with clearer role boundaries, explicit skill contracts (which skills they own versus which they can borrow from other agents), and session protocols for how they share knowledge.

**A shared team brain created:** four files at `Knowledge/team-brain/`:
- `shared-mistakes.md` - patterns that every agent reads at session start
- `shared-patterns.md` - what works, written by any agent, available to all
- `shared-decisions.md` - durable cross-project decisions
- `team-roster.md` - the routing table

**All 13 projects updated:** every project CLAUDE.md now has an `## Active Agents` table that BOOT reads first. When you open a project, the right agents are already routed without any manual loading.

## The Team Brain Design

The insight that made this work was separating shared knowledge from per-agent knowledge.

Per-agent memory (`Knowledge/personas/alex-memory.md`, etc.) holds domain-specific learnings that belong to one persona. Alex's Supabase patterns belong in Alex's memory. Priya's LinkWhisper product decisions belong in Priya's memory.

The shared team brain holds lessons that any agent might encounter and any agent needs to know. The pattern about automated job completion status (a "completed" notification does not mean the job succeeded, always read the output) is something Alex, Sage, and Nova all need. It lives in `shared-mistakes.md`, not in any single agent's memory.

Every agent reads the shared brain at session start. Every agent writes to it when they discover something reusable. The knowledge compounds across agents instead of staying trapped within one.

## The Flat Structure Decision

I chose a flat team with no hierarchy. The alternative was a hierarchical structure with lead agents that delegated to specialists.

Hierarchy adds complexity without adding value when the team size is under 15. A flat team means every agent has clear ownership of one domain, and cross-domain work is handled by naming a primary agent (who leads) and secondary agents (who advise). The routing table in `team-roster.md` handles signal-to-persona matching.

Harper (HR) is the one exception with a constraint attached: Harper recommends but never creates or retires agents. Every agent change requires explicit approval. The purpose is to prevent the team from growing unchecked as new use cases appear. Agent creation has a real cost in maintenance overhead.

## Auto-Invocation via BOOT

The previous system required manually loading agents. You had to remember which agent was relevant, name them in the conversation, and wait for their memory file to be read.

The new system uses BOOT. When you open any project, the CLAUDE.md file for that project lists its active agents. BOOT reads this table and routes to the right persona automatically. The first message in any session already has the right context loaded.

This sounds like a small quality-of-life improvement. In practice it changes the texture of working. You do not spend the first two minutes of a session reconstructing who should be involved.

## What I Learned

**Agents learning in isolation is the primary scaling problem in multi-agent systems.** The fix is a shared knowledge layer that all agents read and write. Individual agent memories plus a shared brain is the right architecture.

**BOOT as a routing layer eliminates activation cost.** If context has to be manually loaded, it will sometimes not be loaded. Automatic routing via project CLAUDE.md means the right agents are always present.

**Harper's 3-evidence rule prevents premature agent creation.** Do not create a new agent until you have seen the same need arise three times across different sessions. Agent proliferation is a real risk.

**Flat over hierarchical for small teams.** Hierarchy adds coordination overhead. At under 15 agents, clear domain ownership and a routing table is enough.

## Related

- [Three Claude Tools, One Vault](three-claude-tools-one-vault.md) — the shared context architecture this team runs on
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer that runs the agents on schedule
- [How I Retired Notion in One Session](how-i-retired-notion.md) — the vault consolidation that gave the team brain a clean foundation
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 redesign of this system
- [Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams](parallel-agents-ai-stack.md) — the parallel agent pattern that this team architecture evolved from
- [Formula Hooks Kill the Metric They Optimize For](formula-hooks-kill-metrics.md) — what happens when an agent executes policy without understanding intent

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

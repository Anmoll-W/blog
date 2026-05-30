<!-- source_session: 2026-04-10_agent-team-v2-launch -->

# Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas

*2026-04-10 · ai-agents, personas, vault · Vault as OS*

> The problem was not that the agents lacked capability. The problem was that they did not know what each other had learned.

## Context

By early April 2026, my Obsidian vault had seven AI agents: an engineer, a product manager for a WordPress SEO plugin, a marketing director, a finance expert, a business analyst, a travel planner, and a vault architect. Each one had an identity file and a memory file. Each one had accumulated real knowledge from real sessions.

The problem was that all of that accumulated knowledge was trapped. Alex (the engineer) had learned that Supabase returns numeric columns as strings and you need to parse them explicitly before doing arithmetic. Maya (the marketing director) had learned that overusing a single hook type causes engagement to collapse even when the data said it was the top performer. These lessons were real and hard-won.

But Zara (the finance expert) started fresh every time. Sam (the business analyst) started fresh every time. When a project touched multiple domains, the relevant agent had to be manually loaded by name. There was no shared understanding. There was no memory that crossed agent boundaries.

I had been building a team of specialists and storing all the institutional knowledge in silos. Every cross-domain session started from scratch on half its context. The problem was not the agents. It was the file structure I had chosen for them.

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

I chose a flat team with no hierarchy. This was an explicit choice, not a default.

The alternative was a hierarchical structure with lead agents that delegated to specialists. That structure feels natural because it mirrors how human teams are organised. The reason I rejected it: a router agent at the top of the hierarchy needs enough knowledge of all the domains below it to make correct routing decisions. That means carrying a summary of every persona anyway. The hierarchy adds a coordination layer without reducing the context load. For a team under 15, that overhead is net negative.

The flat structure trades elegant org charts for something more useful: clear domain ownership and a routing table that any agent can read directly. Cross-domain work is handled by naming a primary (who leads) and secondary (who advise) — no delegation chain required.

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

**On decisions:** the structure of your file system is a product decision, not a housekeeping task. I had seven agents with real accumulated knowledge, and I had made a filing choice that prevented any of that knowledge from being useful across domain boundaries. Whenever a team of specialists is not performing as a team, the first question is not "what are the specialists missing?" It is "what does the structure prevent them from sharing?" The knowledge was there. The routing was missing.

---

**[2026-04-10](../README.md#all-posts)** · [![ai-agents](https://img.shields.io/badge/ai--agents-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![personas](https://img.shields.io/badge/personas-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![prompt-engineering](https://img.shields.io/badge/prompt--engineering-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![obsidian](https://img.shields.io/badge/obsidian-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![knowledge-management](https://img.shields.io/badge/knowledge--management-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [How I Taught My Vault to Read YouTube](youtube-to-vault-pipeline.md) — a new capability the six-persona team now shares: any persona can transcribe a YouTube video into the vault without manual steps
- [Three Claude Tools, One Vault](three-claude-tools-one-vault.md) — the shared context architecture this team runs on
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer that runs the agents on schedule
- [How I Retired Notion in One Session](how-i-retired-notion.md) — the vault consolidation that gave the team brain a clean foundation
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 redesign of this system
- [Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams](parallel-agents-ai-stack.md) — the parallel agent pattern that this team architecture evolved from
- [Formula Hooks Kill the Metric They Optimize For](formula-hooks-kill-metrics.md) — what happens when an agent executes policy without understanding intent
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — the quality layer built on top of this team: a 13th agent whose only job is to stress-test what the other twelve produce
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — what happens when the scheduler that invokes this team silently refuses to run, and the audit that surfaced it
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — the skills layer added on top of this team: installing six external Claude skills and wiring them so agents auto-invoke structured methodology at the right steps
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — what a performance review of this team found: five agents not doing their jobs, and how self-correcting protocols fixed it without a single new hire
- [Four Agents Went RED for Three Weeks. I Cut the Team in Half.](four-agents-went-red.md) — what happened when this 11-agent roster grew to 13 and four agents went RED for three consecutive weeks: a consolidation down to six, with a structured gate block replacing memory as the enforcement layer
- [Upgrading My AI Agent Roster, and the Baseline I Forgot to Save](personas-180-of-180-and-no-baseline.md) — the depth upgrade on the consolidated roster: refusal catalogs, failure-mode lists, dissent protocol, banned-phrase scans, and the missing-baseline lesson about evals without a pre-state to compare against

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

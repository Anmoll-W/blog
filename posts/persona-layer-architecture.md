<!-- source_session: 2026-04-12_agent-management-v3-complete -->

# From Identity Files to Persona Stubs: Redesigning the Vault Agent System

Every session that boots with 12 full agent identity files is a session that started with a context window already under pressure. I ran that system for two months before I sat down to fix it.

The cost was not always visible. But there were sessions where a long-running task would degrade halfway through, or where the agent would lose track of earlier instructions, and I would wonder whether the 24 KB of persona context loaded at the start had anything to do with it. I had built a system optimised for agent readiness and accepted context pressure as the price. I had not stopped to ask whether I needed all twelve agents ready for a session that only needed one.

## The Problem with Version 2

The previous agent system had 12 personas living in an `Agents/[Name]/` folder structure. Each persona had an `identity.md` file with their domain, decision-making style, never-do rules, and accumulated memory. When Claude started a session inside the vault, the boot sequence loaded all of them.

The intent was good. Any task that touched any domain should have the right persona ready. But the execution was expensive. Twelve agents at roughly 2 KB each is 24 KB of context loaded on every session, including sessions that only needed one agent. Most sessions only needed one agent.

The second problem was structural opacity. When you have 12 files with names like Kai, Ren, and Noor, it is not obvious from the name alone what each agent handles. Every time I needed to route a task manually, I had to remember the full mapping.

## What I Evaluated

The redesign started as a question: do I even need agents in the vault anymore, or has the base model gotten good enough that personas are overhead?

After testing, I kept them. Personas still do useful work. They constrain tone, enforce domain-specific never-do rules, and provide consistent framing across sessions. Removing them would have reduced predictability.

So the question became architecture, not existence.

**Approach A: Inline everything in CLAUDE.md.** Put all 12 full persona definitions directly in the root CLAUDE.md. Always loaded, always available. Rejected. The context cost would be worse than the current system, not better.

**Approach B: Single router agent.** One meta-agent whose job is to read the task and decide which persona to activate, then load that persona. Rejected. The router agent would need enough knowledge of all 12 domains to make correct routing decisions, which means it would need to carry a summary of all 12 personas anyway. The savings were illusory.

**Approach C: Per-project CLAUDE.md personas.** Each project gets its own CLAUDE.md with the relevant personas for that project. Rejected. This duplicates persona definitions across eight projects. Any update to a persona requires touching eight files. Maintenance cost scales badly.

**Approach D: Lazy loading with explicit commands.** No personas loaded by default. User explicitly calls a persona with a slash command. Rejected. Too much friction for routine sessions. The system should route itself, not require me to know in advance which agent to invoke.

**Approach E: Stub layer plus on-demand depth.** This is what I built.

## What Approach E Does

The architecture has three layers.

The first layer is the stub layer. Twelve persona stubs live in the vault's root CLAUDE.md. Each stub is roughly 40-50 tokens: agent name, domain label in brackets, a one-line description, a never-do rule, and a deep-load trigger. The full set of 12 stubs comes to under 600 tokens. This is always loaded.

The second layer is the identity layer. Full persona definitions live at `Knowledge/personas/[name].md`. These are only loaded when the session needs depth: teaching, curriculum design, major framework decisions, anything where persona consistency matters over multiple exchanges. The deep-load trigger in the stub tells Claude when to pull the full file.

The third layer is the memory layer. Each persona has a companion file at `Knowledge/personas/[name]-memory.md` that stores accumulated learnings from past sessions. This is loaded when the task requires continuity with prior decisions.

A SessionStart hook ties it together. When a session begins, the hook reads the task description, matches signal words against a routing table, and activates the right stub. For most sessions, the stub is sufficient. The full identity and memory files are only loaded when the routing logic or the user explicitly triggers a deep-load.

Two agents were renamed during the rebuild to make the stub layer more readable. Kai became Sage (domain: AI Architect and Technical Educator). Ren became Lux (domain: Creative Designer). Arbitrary names require memory. Descriptive names make the routing table legible at a glance. Every agent also got an explicit domain label in brackets, so the stub reads like `[Platform · Engineering]` rather than requiring me to remember what the name maps to.

Three stateless task subagents were added at `~/.claude/agents/`: `@code-review`, `@draft-post`, and `@qa-check`. These are not personas. They are task executors with no memory or identity. They run once and stop. Keeping them separate from the persona layer prevents the routing logic from conflating character consistency with task specialization.

## The Key Insight

Context window cost and routing depth are in tension, and the right architecture separates them into different layers instead of collapsing them together.

Version 2 treated routing and depth as the same problem. If you might need a persona, load it fully. Approach E treats them as different: routing needs a stub, depth needs a file, and most sessions only need routing.

The practical result is that routine sessions now boot with under 600 tokens of persona context instead of 24 KB. Sessions that need depth load one or two full identity files on demand, not all twelve upfront. The system is faster to boot, cheaper on context, and easier to maintain because persona updates touch one file rather than eight.

The old `Agents/` folder is archived at `Assets/archive/agents-v2/`. Not deleted, because the identity files contain accumulated memory that would be expensive to reconstruct. Preserved, but no longer in the active load path.

## What I Learned

Stub layers are underused in agent design. Most systems I have seen either load everything or load nothing. The middle ground — always-load a routing summary and on-demand-load the full definition — is the right default for any system with more than three or four agents.

Descriptive naming pays compound interest. Arbitrary names (Kai, Ren, Noor) are fine when you designed the system and have it memorised. They are friction for every future session where you need to reason about routing. Domain labels in the stub cost almost nothing and make the routing table self-documenting.

Approach rejection is part of the design artifact. I evaluated five approaches and rejected four before landing on Approach E. That rejection history is worth keeping. When someone asks why the system does not use a single router agent, the answer is in the decisions file, not lost to memory.

On decisions: the right question when a system feels slow or expensive is not "how do I make this faster?" It is "what am I loading that I do not need?" I had been treating agent readiness as a binary — either loaded or not loaded. The decision that changed the system was separating routing depth from context depth. Most sessions need routing. Few sessions need the full identity. Building a layer for each, instead of collapsing them together, is a design pattern that applies well beyond agent systems. Any time the cost of "ready for everything" is being paid on sessions that only need one thing, the right move is a stub layer, not a bigger context window.

---

**[2026-04-12](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![prompt-engineering](https://img.shields.io/badge/prompt--engineering-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [Three Claude Tools, One Vault](three-claude-tools-one-vault.md) — the original shared context architecture this builds on
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the broader automated system Approach E fits inside
- [How I Retired Notion in One Session](how-i-retired-notion.md) — the vault consolidation that preceded this redesign
- [Building an AI Agent Team](building-an-ai-agent-team.md) — the v2 system this redesign replaced
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — Vera, persona #13, added as the evaluation layer on top of this stub architecture
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — the next layer added on top of this architecture: external skills wired to each persona so agents auto-invoke structured methodology without being asked
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — the audit layer: what a performance review of the personas built here found, and the self-correcting protocols added to fix it

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

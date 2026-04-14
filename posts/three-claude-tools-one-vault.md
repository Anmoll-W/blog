<!-- source_session: 2026-03-25_claude-dispatch-obsidian-vault -->

# Three Claude Tools, One Vault: The Architecture Behind the System

Before I had automated agents and LaunchAgents running on schedule, I had a simpler question: how do I make all three Claude interfaces — mobile, desktop, and development — share the same brain?

The answer was one Obsidian vault and a context cascade.

## Context

I use Claude in three modes depending on where I am and what I am doing. Claude Dispatch on my phone handles quick captures and lookups away from the desk. Claude Desktop handles research, writing, and knowledge work. Claude Code handles everything in development — reading files, writing code, running scripts, managing the vault itself.

Each tool is powerful on its own. The problem is they start from zero each session. No memory of what was decided last week. No knowledge of what projects are in flight. Each conversation feels like the first.

The solution was not to fix each tool separately. It was to build one context system that all three tools read from.

## The Architecture

Every `.md` file I write makes all three tools smarter. Here is how.

**Context cascade.** The vault has a root `CLAUDE.md` at the top level. Every project folder has its own `CLAUDE.md`. When Claude opens a file in any tool, it reads the nearest `CLAUDE.md` first, then the root. This gives every conversation the right context without me explaining it from scratch.

**Folder-level skills.** Skills live inside the folder they serve. A travel planner skill lives inside the travel project. An inbox processor skill lives at the vault root. The skill is co-located with its domain, not stored in a separate "skills" folder that drifts out of sync.

**Inbox processing pattern.** Daily notes use tags: `#process`, `#task`, `#idea`. When I tag something in a daily note, Claude sorts it to the right destination via a skill that runs on the vault. The tags are the routing layer. The skill is the executor.

**Nick Milo dossier prompt.** One reusable prompt generates an "about me" context block — my role, active projects, goals, constraints. Every new conversation reads this first. Claude knows who I am before I say anything.

## What This Enabled

The dispatch/desktop/code split works better with this in place. I can send a quick capture on my phone ("add this to the travel project backlog") and Claude Desktop will have context on it the next time I open that folder. Claude Code will read the same decisions and past mistakes files before touching anything.

The vault is not a notes app anymore. It is a shared memory layer that all three tools read from.

## What I Learned

Reliability matters more than capability on mobile. Dispatch works well for inbox triage, lookups, and meeting prep. Multi-step file organization and cross-app chains fail unpredictably. Keep mobile tasks simple and single-step.

The insight that unlocked this system: context does not belong in each tool. It belongs in a shared file system that all tools read. Once that shift happened, the tools stopped feeling disconnected.

---

**[2026-03-25](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![obsidian](https://img.shields.io/badge/obsidian-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer built on top of this architecture
- [How I Retired Notion in One Session](how-i-retired-notion.md) — consolidating all knowledge into one vault
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the coordination layer
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 redesign of the agent architecture

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

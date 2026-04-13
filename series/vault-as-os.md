# Series: Vault as OS

How I turned an Obsidian vault into an automated knowledge system powered by AI agents.

The system today runs three automated agents on schedule, routes every daily note capture to its destination without manual sorting, coordinates an 11-persona AI team across 14 projects, and compresses its own knowledge base weekly to stay lean as it grows.

This series documents how each layer was built, what broke along the way, and what the architecture looks like now.

> **System:** Built around an Obsidian vault. See [Anmoll Wadhwa](https://github.com/Anmoll-W) for context on the full toolchain.

## Posts in This Series

0. [Three Claude Tools, One Vault: The Architecture Behind the System](../posts/three-claude-tools-one-vault.md)
   The starting point: how three Claude interfaces (mobile, desktop, code) share one vault as a common memory layer. Covers the context cascade, folder-level skills, and the inbox processing pattern.

1. [From Manual to Automatic: How I Built a Vault OS That Runs Itself](../posts/vault-os-that-runs-itself.md)
   The automation layer: three LaunchAgents running morning briefings, inbox processing, and weekly reviews on schedule. Covers why CronCreate failed and why macOS LaunchAgents were the right answer.

2. [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](../posts/building-an-ai-agent-team.md)
   The coordination layer: replacing isolated agents with a flat team that shares a knowledge brain. Covers the shared team brain architecture, auto-invocation via BOOT, and how Harper prevents agent proliferation.

3. [How I Retired Notion in One Session](../posts/how-i-retired-notion.md)
   The consolidation: migrating all Notion content to the vault in one session, deleting tasks files, and fixing 23 stale path references. Why everything that Claude Code needs to read must live as Markdown.

4. [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](../posts/persona-layer-architecture.md)
   The v3 redesign: replacing 12 full identity files loaded on every session with a stub layer plus on-demand depth loading. Covers the five approaches evaluated and why Approach E won.

5. [The Eval Agent: Adding a Quality Gate to an AI Workflow](../posts/the-eval-agent.md)
   The evaluation layer: adding a 13th persona whose only job is to stress-test output. Covers the rubric system, why producing agents cannot eval their own work, and the blind-spot question pattern.

## What Is Coming

- The knowledge compression system: how the vault keeps its own files lean with weekly archival and monthly summarization
- The session-to-blog pipeline: turning vault session notes into public writing automatically

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*

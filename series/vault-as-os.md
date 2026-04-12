# Series: Vault as OS

How I turned an Obsidian vault into an automated knowledge system powered by AI agents.

The system today runs three automated agents on schedule, routes every daily note capture to its destination without manual sorting, coordinates an 11-persona AI team across 14 projects, and compresses its own knowledge base weekly to stay lean as it grows.

This series documents how each layer was built, what broke along the way, and what the architecture looks like now.

## Posts in This Series

1. [From Manual to Automatic: How I Built a Vault OS That Runs Itself](../posts/2026-04-05-vault-os-that-runs-itself.md)
   The automation layer: three LaunchAgents running morning briefings, inbox processing, and weekly reviews on schedule. Covers why CronCreate failed and why macOS LaunchAgents were the right answer.

2. [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](../posts/2026-04-10-building-an-ai-agent-team.md)
   The coordination layer: replacing isolated agents with a flat team that shares a knowledge brain. Covers the shared team brain architecture, auto-invocation via BOOT, and how Harper prevents agent proliferation.

## What Is Coming

- The knowledge compression system: how the vault keeps its own files lean with weekly archival and monthly summarization
- The session-to-blog pipeline: turning vault session notes into public writing automatically

---

*[All posts](../README.md)*

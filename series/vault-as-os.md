# Series: Vault as OS

How I turned an Obsidian vault into an automated knowledge system powered by AI agents.

The system today runs seven automated agents on schedule, routes every daily note capture to its destination without manual sorting, coordinates an 11-persona AI team across 14 projects, and compresses its own knowledge base weekly to stay lean as it grows.

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

6. [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](../posts/second-automation-layer.md)
   The second automation layer: expanding from three scheduled agents to seven, adding Chrome-gated analytics sync, daily task migration, Monday content dispatch, and quality gates on code review and blog publishing. Covers the sleep-gap bug, guard clause patterns, and why monthly operations belong inside weekly runners.

7. [My AI Agents Had Identity. They Needed Methodology.](../posts/wiring-claude-skills-to-agents.md)
   The skills layer: installing six external Claude skills from GitHub and wiring them into the agent system. Covers the auto-invoke vs. situational distinction, how each of the three stateless agents changed, and why the QA agent had been starting from the wrong assumption.

8. [My AI Agents Were Running. They Just Weren't Working.](../posts/self-correcting-agents.md)
   The self-correction layer: what happens when you audit a 13-agent system and find five aren't doing their jobs. Covers the difference between a protocol and a reminder, how self-correcting behaviors are baked into agent identity, and why "operational" is a claim that needs evidence.

9. [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](../posts/agents-that-dont-learn.md)
   The compounding layer: how verbal self-reflection (Reflexion Block), a user profile mined from 60 sessions, memory decay flags, path-scoped rules, and explicit agent frontmatter turned a system that accumulated sessions into one that accumulates intelligence.

## What Is Coming

- The knowledge compression system: how the vault keeps its own files lean with weekly archival and monthly summarization

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*

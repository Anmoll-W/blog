---
title: "How I Retired Notion in One Session"
date: 2026-04-08
tags: [vault, systems, obsidian, ai]
series: vault-as-os
series_order: 3
source_session: 2026-04-08_vault-audit-notion-migration
related:
  - 2026-03-25-three-claude-tools-one-vault.md
  - 2026-04-05-vault-os-that-runs-itself.md
  - 2026-04-10-building-an-ai-agent-team.md
  - 2026-04-12-persona-layer-architecture.md
---

# How I Retired Notion in One Session

For two years I maintained two knowledge systems in parallel. Notion held the structured, long-form content: project specs, roadmaps, strategy docs. Obsidian held session notes, quick captures, and the vault architecture. Every meaningful session created friction: where does this go?

One session ended that split permanently.

## What the Problem Actually Was

The problem was not that Notion was bad. It was that two systems create a routing tax on every piece of information. Before writing anything I had to ask: is this a Notion thing or a vault thing? Sometimes I got it wrong and spent time later looking in the wrong place.

More importantly: Claude Code cannot read Notion. Everything I wanted the AI system to know had to live in the vault as plain Markdown. Any knowledge sitting in Notion was invisible to the tools I rely on most.

## What the Session Did

The session ran in several phases.

**Vault hygiene first.** The Obsidian theme was throwing a QuickAdd crash. Fixed that before touching anything structural. A broken capture tool would have made the migration harder.

**Flattened the personal projects.** The personal projects folder had grown into a nested structure. Flattened it into three clean top-level projects: Finance, Health, Travel. Each gets its own `CLAUDE.md` and `knowledge/` folder. No nesting.

**Migrated all Linkwhisper Notion content to vault.** Seven knowledge files, all decisions, all tasks. Each became a Markdown file in the correct project subfolder. The vault became the single source of truth for everything in flight.

**Retired the tasks files.** Every project had a `tasks.md`. All of those were deleted. Tasks now live in daily notes with inline project tags. A Dataview query on the home dashboard shows all open tasks by tag. One source, no duplication.

**Fixed 23 stale path references.** After moving files, eight vault-level files had silent broken links: the home dashboard, several CLAUDE.md files, two skills. None of these would have thrown errors — they just would have silently routed wrong. Searched for every old path reference and updated them.

## Why Tags Replace Category Folders

The old system used numbered folders as categories: `04 Linkwhisper`, `05 Personal`, `06 Finance`. Every time the structure changed, paths broke.

The new system uses tags in daily notes: `#linkwhisper`, `#finance`, `#health`, `#travel`. The Dataview home dashboard queries by tag. No paths, no folders to maintain. Adding a new project means adding a new tag to the query, not restructuring the folder tree.

## What Did Not Make the Cut

Voice note processing. The inbox processor handles text tags cleanly. Voice notes require a transcription layer that is not worth building until the text system is solid.

Finance tracking. Zara (the finance persona) needs structured data, not prose notes. That will stay in a dedicated finance tool until there is a reason to move it.

## What I Learned

The migration did not take long because most of the Notion content was already stale. Two years of project docs, meeting notes, strategy drafts — almost none of it was relevant to current work. The act of migrating forced a cleanup that I had been deferring.

The rule that emerged: any knowledge that needs to be readable by Claude Code must live as Markdown in the vault. Everything else is optional. Notion had become a graveyard for context that the system could not use.

One more pattern: the stale path problem is invisible until it breaks something. After any vault restructure, run a search for old folder names before moving on. Silent misdirections are worse than obvious errors.

## Related

- [Three Claude Tools, One Vault](2026-03-25-three-claude-tools-one-vault.md) — why the vault became the shared memory layer in the first place
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](2026-04-05-vault-os-that-runs-itself.md) — the automation layer that now runs on top of this structure
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](2026-04-10-building-an-ai-agent-team.md) — the agent coordination layer that runs on top of this consolidated vault
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](2026-04-12-persona-layer-architecture.md) — the v3 agent redesign that depends on this structure

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

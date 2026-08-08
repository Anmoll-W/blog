<!-- source_session: 2026-04-08_vault-audit-notion-migration -->

# How I Retired Notion in One Session

*2026-04-08 · vault, systems, obsidian · Vault as OS*

For two years I maintained two knowledge systems in parallel and told myself the split made sense. Notion for the structured, permanent work — project specs, roadmaps, strategy docs. Obsidian for session notes, daily captures, and the vault itself.

What I did not account for was the cost that compounded quietly in the background: every time I built something that needed to know about my projects, it could only read one of the two systems. Claude Code can read Markdown files in a vault. It cannot read Notion. Every spec, every decision, every roadmap that lived in Notion was invisible to the tools I was relying on most.

I had been building an AI-powered knowledge system on top of half my knowledge.

## What the Problem Actually Was

The problem was not that Notion was bad. It was that two systems create a routing tax on every piece of information. Before writing anything I had to ask: is this a Notion thing or a vault thing? Sometimes I got it wrong and spent time later looking in the wrong place.

The decision to consolidate was a decision to define a rule: any knowledge that needs to be readable by Claude Code must live as Markdown in the vault. That rule made the choice obvious and eliminated the routing question permanently. Notion was not retired because it was worse. It was retired because it was on the wrong side of that rule, and maintaining a second system to house knowledge my tools could not use was not a cost I was willing to keep paying.

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

The migration did not take long because most of the Notion content was already stale. Two years of project docs, meeting notes, strategy drafts — almost none of it was relevant to current work. The act of migrating forced a cleanup that I had been deferring. I had been maintaining a second system to house documents I was not using, and the cost had been invisible because neither system told me the other existed.

The rule that emerged: any knowledge that needs to be readable by Claude Code must live as Markdown in the vault. Everything else is optional. Notion had become a graveyard for context that the system could not use.

One more technical pattern: the stale path problem is invisible until it breaks something. After any vault restructure, run a search for old folder names before moving on. Silent misdirections are worse than obvious errors.

On the decision side: a two-system architecture is worth maintaining only if both systems serve the same consumers. The moment you build a workflow that can only read one of them, you have made a silent decision about which system actually matters. I had made that decision when I started relying on Claude Code — I just had not acted on it. The session that retired Notion was not about Notion. It was about catching up to a decision I had already made in practice but not yet in structure.

---

**[2026-04-08](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![obsidian](https://img.shields.io/badge/obsidian-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS](../series/vault-as-os.md)

## Related

- [Three Claude Tools, One Vault](three-claude-tools-one-vault.md) — why the vault became the shared memory layer in the first place
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer that now runs on top of this structure
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the agent coordination layer that runs on top of this consolidated vault
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 agent redesign that depends on this structure

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

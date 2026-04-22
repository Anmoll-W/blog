<!-- source_session: 2026-04-22_kanban-board-hub-setup -->

# The Dashboard Was Lists. The Hub Is a Board.

*2026-04-22 · vault, obsidian, kanban, systems, productivity · Vault as OS*

Home.md had everything.

Every project's open tasks, pulled live via Dataview. Inbox items. Recent sessions. The LinkedIn queue. Neatly organized into sections with headings for each project. It loaded in under a second and was technically correct at all times.

I opened it every morning and understood nothing.

Not because the data was wrong. Because a vertical list of tasks grouped by project does not answer the question I actually had: what is in progress right now, and what is stuck?

---

## The Problem With Lists

Home.md was structured like a report. LinkWhisper tasks. LinkedIn tasks. Blog tasks. Scroll down for Finance. Scroll further for Personal. Every task was there if I looked for it. None of the relationships between tasks were visible — what was actively being worked on, what was queued but not started, what had been sitting untouched for three weeks.

The information was there. The visibility was not.

A dashboard that requires effort to read is not a dashboard. It is a filing cabinet with a better interface.

What I needed was a board.

---

## Installing the Plugin

The Obsidian Kanban plugin by mgmeyers renders markdown files as visual boards. Each `##` heading becomes a column. Each list item becomes a card. The underlying file is still plain markdown — readable in any editor, backed by git, not locked into a proprietary format.

Installation is manual if you want to skip the in-app browser: download `main.js`, `manifest.json`, and `styles.css` from the GitHub release, drop them into `.obsidian/plugins/obsidian-kanban/`, and add the plugin ID to `community-plugins.json`.

One thing that took a debug pass to find: the `%% kanban:settings %%` block at the bottom of the board file requires raw JSON inside backtick fences with no language specifier. Adding `json` after the opening backticks causes a parse error on load. The plugin fails silently in a way that makes it look like a corrupted file rather than a settings syntax issue.

---

## The First Design Decision: What Are the Columns?

The obvious choice is columns by project: a LinkWhisper column, a LinkedIn column, a Blog column. Open the board, look at a column, see every task for that project.

The problem is that columns by project answer "what does each project need" but not "what is actually happening." You end up with eight columns of backlog tasks and no ability to see, at a glance, what is in flight versus what is waiting versus what nobody has touched in a month.

The better choice is columns by status.

Five columns: **Backlog** (defined but not started), **Up Next** (committed for this week), **Active** (being worked on now), **Blocked** (waiting on something external), **Done This Week** (pruned every Sunday by the weekly-review runner).

The Active column has a hard cap of three cards. If something new needs to move to Active, something already there must move to Done or Blocked first. Without a cap, Active becomes a second Backlog — just one with a more optimistic name.

Status columns answer the daily question. Project columns answer the inventory question. The daily question is more important.

---

## Project Identity Without Project Columns

Switching to status columns creates a new problem: how do you know which project a task belongs to?

The solution is tags rendered as colored badges.

In the Kanban plugin settings block, each tag gets a color: `#linkwhisper` is blue, `#linkedin` is indigo, `#blog` is green, `#pmdojo` is orange, `#chalotrip` is teal, `#finance` is gold, `#automation` is gray, `#decisions` is red. A card that reads "TroubleShooter spec" with a blue `#linkwhisper` badge is immediately identifiable without needing a project column.

The tags are configured in the `%% kanban:settings %%` JSON block:

```json
{
  "tag-colors": [
    {
      "tagKey": "linkwhisper",
      "color": "rgba(59,130,246,1)",
      "backgroundColor": "rgba(219,234,254,0.6)"
    }
  ]
}
```

Note that `tagKey` omits the `#` prefix.

Nine project tags. Nine colors. You can identify the project on any card in under a second without reading the text.

---

## Every Card Is a Link

The second structural decision was that no card should be a dead end.

In the Kanban plugin, card text can contain wikilinks. If a card reads `[[Projects/Linkwhisper/TroubleShooter/prd|TroubleShooter spec]]`, clicking the card title in the board view opens the spec directly.

Every card in the master board links to something: the project PRD, the relevant knowledge file, the specific draft, or the project index. The board becomes a navigation layer over the vault's knowledge structure, not just a task list floating in isolation.

This matters because the board should not be the place where information lives. It is the place where you decide what to work on and get to the information that already exists. Cards are pointers, not containers.

---

## Two Layers, No Sync

The instinct when building a task board is to make it the single source of truth for all tasks. Every task goes on the board. The board is authoritative.

This breaks quickly.

The problem is that not all tasks are the same kind of thing. "Write the TroubleShooter spec" is a project-level task. It has a scope, a deliverable, a connection to a PRD, and probably a few days of work attached to it. It belongs on the board.

"Draft today's LinkedIn post" is a day-level task. It is about today. It has a slot in the posting schedule. It will be done in a few hours. It belongs in the daily note.

Trying to put both on the same board creates maintenance debt. The day-level tasks clutter the board and make the project-level signal harder to see. The project-level tasks are too vague for a daily note. And anything that has to be manually synced between two places will drift until you stop trusting one of them.

The system runs two layers intentionally:

- **Board**: strategic tasks — specs, campaigns, features, decisions. Anything that spans multiple sessions.
- **Daily notes**: operational tasks — today's post, today's meeting, today's fix. Anything that is done by end of day.

No sync. The inbox-processor automation that runs every night handles routing from daily notes to their destinations. It does not touch the board. The board is maintained by hand, which means every card on it is something you consciously chose to track.

---

## The Hub Column

Retiring Home.md created a gap: where do the navigation links go?

Home.md had links to active-context, decisions, sessions, the LinkedIn queue. Useful quick-access items that were not tasks.

The solution was a pinned Hub column — the leftmost column of the master board, containing only navigation cards:

- Daily Notes folder
- Active Context
- Decisions
- Sessions
- LinkedIn Board
- LinkWhisper Board
- Automation Health

Not tasks. Not cards to complete. Wikilinks presented as cards in a column that never moves and never needs to be pruned. The board opens and the navigation layer is there on the left, stable, before you look at any of the task columns.

Home.md now contains one line: `→ [[Board|📋 Open Board]]`.

---

## Sub-Boards for Complex Projects

The master board tracks one or two cards per project — the most active items. It is not meant to hold every task for every workstream.

For projects with three or more concurrent workstreams, a dedicated sub-board lives inside the project folder.

The criteria matter: a project with one active workstream does not need a sub-board. It adds navigation overhead without adding clarity. The threshold is three concurrent streams, because that is where a flat backlog starts losing information about how tasks relate to each other.

Two projects currently have sub-boards:

**LinkWhisper** has four workstreams running in parallel: a user-facing troubleshooting widget (WP REST API + Vite/Preact), a done-for-you service, a website conversion project, and a support dashboard. The sub-board uses secondary tags — `#troubleshooter`, `#dfy`, `#cro`, `#supportdash` — so each card shows both its status (via column) and its workstream (via tag). The master board links to it from the Hub column.

**LinkedIn** has a content pipeline with distinct stages: Ideas (captured angles, not drafted), Drafting (actively being written), Ready (complete, scheduled), Published. Each card is a specific post linked directly to its draft file. This replaces manually scanning a folder of markdown files to find what is ready to publish.

---

## Maintenance Built In

A Kanban board that requires manual weekly cleanup gets abandoned.

The weekly-review runner — a shell script that calls Claude every Sunday evening — was extended with one instruction: after the review is complete, read every Done column in every board, move the completed cards into a dated archive file at `Knowledge/sessions/YYYY-MM-DD-weekly-wins.md`, and replace the Done column with an empty header.

The board clears itself. The wins are preserved. The archive builds over time as a record of what shipped each week.

Backlog cards that have not moved in 30 days are flagged during the weekly review as stale and surfaced for a pursue-or-kill decision. A board where old tasks stay indefinitely becomes a backlog graveyard — things you meant to do that you will never do, taking up visual space alongside things you are actually doing.

---

## What the Board Actually Changed

The board did not add information that did not already exist. The tasks were in daily notes. The projects were in their folders. The specs were written.

What changed was the question the system could answer in under five seconds.

Before: open Home.md, scroll through project sections, mentally reconstruct what is active versus backlog versus stuck.

After: open Board.md, look at the Active column, see three cards with colored project badges, each one a clickable link to its detail file.

The board does not manage the work. It shows the shape of the work — which is the thing a system needs to do before you can decide what to do next.

---

## Related

- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — how enforcement-by-infrastructure replaced enforcement-by-documentation, the same principle applied to session behavior
- [The Vault Was Organized. The Files Were Not.](vault-structural-drift.md) — the hygiene layer that keeps the project structure the board navigates into clean
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer that handles nightly routing and Sunday cleanup, which the board's maintenance now depends on
- [The Agents Were Ready. The Coordination Was Not.](skill-chaining-agent-orchestration.md) — how the same "visibility without duplication" principle was applied to multi-agent coordination

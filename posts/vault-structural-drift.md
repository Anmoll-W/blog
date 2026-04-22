<!-- source_session: 2026-04-21_vault-organizer-automation -->

# The Vault Was Organized. The Files Were Not.

*2026-04-21 · vault, automation, ai, obsidian, systems · Vault as OS*

I ran a dry test of the new vault-organizer and watched it move 31 files.

Not one or two stray notes. Thirty-one files in the wrong location, accumulated silently across six weeks of daily sessions.

The vault had a clear structure. Every project had the same anatomy: a CLAUDE.md, an index.md, a knowledge/ folder, and a sessions/ subfolder. The documentation was thorough. The naming conventions were consistent. And none of it had been enforced.

That is the gap this post is about: the difference between a structure that is documented and a structure that runs.

---

## What Was Drifting

The violations followed three patterns.

**Session notes at project roots.** Every time a session note got saved during a fast-moving build, it landed at the project root — `Projects/Linkwhisper/2026-04-15_pricing-cro-audit.md` — instead of `Projects/Linkwhisper/knowledge/sessions/`. The session happened. The note was created. It just ended up one level above where it belonged.

**Loose files in sub-project folders.** As sub-projects got created (Website, TroubleShooter, Done-For-You), miscellaneous files that were not an index or CLAUDE.md landed directly at the sub-project root instead of inside a knowledge/ subfolder.

**Missing index.md stubs.** Eight sub-project folders had no index.md at all. They existed, they contained files, but there was no entrypoint — which means Obsidian Graph View showed them as orphaned clusters and any automated tool that expected an index would either error or create one.

None of these were catastrophic. Nothing was broken. But they were invisible to me, and they were compounding every session.

---

## The Auto-Move Decision

The first design question was: should the automation move files, or alert on violations and let me fix them manually?

Alert-only feels safer. It preserves human judgment. The automation raises its hand, and the human decides.

The problem with alert-only is that it adds a step back into a workflow I was trying to remove steps from. The vault-organizer would run nightly at 9:30 PM, generate a list of violations, and then — nothing would happen unless I read the log and acted on it. Two out of five days I would. The other three, the list would sit there, the violations would accumulate, and the next weekly review would surface them as a pile I had been deferring.

That is the same failure mode as running agents manually: systems that require memory to use do not stay used.

The alternative was auto-move with a safety log. Every move gets recorded in an append-only log with the original path, the new path, and whether any wikilinks were patched. Nothing is ever deleted. If something moves incorrectly, the log shows where it went. The cost of an incorrect move is low; the cost of a structure that degrades silently is compounding.

Chose auto-move.

---

## How It Works

The vault-organizer is a skill file — a set of step-by-step instructions that a Claude Code session follows when invoked. It runs as step 2 of the existing inbox-processor runner at 9:30 PM IST, after a 60-second sleep.

**Step 1: Find session files outside knowledge/sessions/.**

```bash
find Projects \( -name "20[0-9][0-9]-[0-9][0-9]-[0-9][0-9]_*.md" \
  -o -name "20[0-9][0-9]-[0-9][0-9]-[0-9][0-9]-*.md" \) \
  | grep -v "/knowledge/sessions/"
```

For each result, identify the parent project, construct the target path, check for path-based wikilinks, patch any that exist, then move.

**Step 2: Find loose files at project and sub-project roots.**

Files that are not CLAUDE.md, index.md, prd.md, MOC.md, README.md, or a small set of recognized root-level files → check if they are already inside a knowledge/ folder → if not, move to `<same-parent>/knowledge/<filename>`.

**Step 3: Create missing index.md stubs.**

Any folder under Projects/ at depth 1-3 with no index.md gets a minimal stub:

```markdown
---
type: index
updated: [date]
---

# [Folder Name] — Index

*Auto-created by vault-organizer. Fill in project details.*

[[Projects/[parent]/index|← [Parent]]]
```

**Step 4: Write the report.**

Append a dated section to `Knowledge/vault-organizer-log.md` — moved files, stubs created, wikilinks patched, errors. If nothing was moved or created: a one-line clean entry.

---

## No New Infrastructure

One decision worth naming explicitly: no new LaunchAgent.

The vault already runs an inbox-processor at 9:30 PM IST every day. That runner invokes a Claude Code session to process daily note captures. Vault-organizer runs in the same session window, using the same tool permissions, after the inbox processing is done.

Adding a new LaunchAgent would mean a new plist file, a new runner script, a new log file, and another cron slot to track. The vault-organizer does not need its own session — it needs 10-15 minutes of attention after the inbox-processor finishes.

Piggybacking is simpler, and the inbox-processor slot was already being used for vault maintenance. Adding a second maintenance pass in the same slot keeps the system comprehensible.

---

## The Wikilink Patching Problem

Obsidian resolves wikilinks in two ways: short-name (`[[filename]]`) and path-based (`[[Projects/X/knowledge/sessions/filename]]`). Short-name links survive file moves automatically — Obsidian tracks the file regardless of where it lives. Path-based links break.

Before moving any file, the vault-organizer greps for path-based references:

```bash
grep -rn "[[Projects/Linkwhisper/2026-04-15_pricing-cro-audit]]" --include="*.md" .
```

If references are found, it patches each one to the new path before moving. If no references are found, the move is safe as-is.

In practice, most session notes had no path-based references — short-name links are the default in Obsidian. But four files from the LW restructure had path-based links in their parent index files, and those needed patching before the move. The grep-before-move pattern handled those correctly.

---

## The Visibility Chain

Running a nightly automation that modifies files silently is only safe if you can see what happened.

Three layers of visibility:

**vault-organizer-log.md** — append-only, dated sections. Every move, every stub, every wikilink patch. The ground truth for what the organizer did.

**automation-health.md** — after each run, the inbox-processor runner writes a single line: `[timestamp] vault-organizer ✅` or `❌` or `⚠️ TIMEOUT`. The morning-briefing reads this file and surfaces any non-green status.

**weekly-review** — has an automation health check section that scans the past 7 days of automation-health.md entries. Any failure in the past week appears in the weekly review and gets flagged.

If the vault-organizer fails silently for three days, I will see it in the morning brief on day one and again in the weekly review. Nothing stays hidden.

---

## What I Learned

**Technical:** Placement rules defined in documentation do not enforce themselves. The vault had explicit, well-documented structure conventions for months. Thirty-one files ignored them — not because the conventions were unclear, but because nothing was checking. Automated enforcement is not redundant to documentation; it is what makes documentation real.

**Decision-making:** The question "should this be manual or automated?" almost always resolves to: how many times will this action need to happen? One-off cleanup is manual. Ongoing structural hygiene is automated. The vault grows every day. The organizer runs every day. Those two clocks need to tick together.

---

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the first automation layer: three LaunchAgents running on schedule. The vault-organizer follows the same pattern but runs inside an existing session window instead of its own.
- [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](second-automation-layer.md) — expanding the automation layer from three to seven scheduled agents. The vault-organizer extends this to an eighth routine.
- [The Agents Were Ready. The Coordination Was Not.](skill-chaining-agent-orchestration.md) — how the skills registry and orchestration contracts were built. The vault-organizer skill follows the same skill file pattern.
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — the previous session's work on the agent system. The vault-organizer was built in the session immediately after this.
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — the silent failure that stopped the first generation of LaunchAgents. Directly relevant context for anyone adding scheduled vault automation on macOS.
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the same enforcement gap applied to Claude Code session start: documented rules that ran only when remembered, fixed by moving them into a hook that fires every session.
- [The Dashboard Was Lists. The Hub Is a Board.](kanban-board-as-project-hub.md) — the navigation layer built on top of the structure this post enforces. The Kanban hub links directly into the project folders the vault-organizer keeps clean.

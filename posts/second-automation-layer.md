<!-- source_session: 2026-04-15_vault-agent-automation-buildout -->

# Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7

*2026-04-15 · vault, automation, ai, macos, launchagent, systems · Vault as OS*

The system was working. Three LaunchAgents fired on schedule: morning briefing at 8:30 AM, inbox processing at 9:30 PM, weekly review on Sundays. The vault was getting smarter without manual effort.

The problem was the gaps. Everything outside those three windows still required me to notice, remember, and act.

I had analytics data landing in Chrome that never made it to the vault. Completed tasks sitting in daily notes that never migrated to project logs. LinkedIn drafts that were finished but had no routing system to surface them on Monday mornings. A code review step that only happened when I remembered to ask for it.

The second automation layer closed those gaps. This post documents what was built, what broke during the build, and the decision logic behind each piece.

## Context

The prior state of the system: three macOS LaunchAgents running shell scripts that called `claude -p` against the vault. Each script had a specific instruction prompt, a tool allowlist, and a reference to the vault directory. The outputs were appended to the daily note or committed to session files.

This architecture proved itself over three weeks. The right question at this stage was not "can we add more?" but "what maintenance costs do I actually pay by hand every week that a headless agent could own?"

The audit produced seven items in three categories:

**Analytics and capture** — data that existed somewhere but was not reaching the vault automatically  
**Weekly maintenance** — tasks that the weekly review was doing in bulk on Sundays but should run daily to stay lean  
**Content and quality gates** — review steps that happened only when remembered

## What Was Built

### LinkedIn Analytics Sync (daily at 9:07 AM IST)

LinkedIn analytics require a browser session. The agent uses a Chrome extension that reads page content without user interaction.

The scheduling decision here required a guard clause. The agent needs Chrome to be open and logged in to LinkedIn. If it runs at 9:07 AM and Chrome is not open, it should not crash or produce an error — it should log one line and exit cleanly.

The solution is a process check at the start of the script:

```bash
if ! pgrep -x "Google Chrome" > /dev/null 2>&1; then
  echo "Chrome not open — skipping analytics sync"
  exit 0
fi
```

If Chrome is running, the agent navigates to the analytics page, extracts the numbers, and appends them to the vault's performance log. If Chrome is not running, nothing happens. The 9:07 AM time was chosen deliberately — slightly offset from the 9:00 AM morning briefing to avoid two agents competing for resources.

### Done-Tasks Migrator (daily at 10:00 PM IST)

The weekly review skill had a step that moved completed tasks from daily notes to project-level done-tasks files. Running this once a week meant the migration lagged by up to six days. By Sunday, a task completed on Monday had been sitting in the wrong file for the entire week.

Moving this to a daily agent at 10:00 PM IST (after the inbox processor at 9:30 PM) means every completed task is in the right file within 24 hours. The agent reads today's daily note, identifies `- [x]` items, matches each one to a project based on its tags, and appends it to the relevant project file under a monthly header.

### Draft Post Dispatcher (every Monday at 10:00 AM IST)

Every week I had drafted LinkedIn posts that were sitting in the vault unpublished. The gap was not writing — it was surfacing. On Monday mornings I would forget to check the draft bank before deciding what to post.

The dispatcher solves this by running once a week before the day starts. It reads every draft in the LinkedIn posts folder, checks their status, ranks them by content pillar and recency, and writes a `## LinkedIn Queue` section to that day's note. When I open my vault on Monday, the queue is already there.

### Expanded Sunday Bundle

The Sunday bundle previously ran two sequential sessions: a finance check-in followed by a full weekly review. Both were critical — finance first, then the structured week assessment.

Three maintenance tasks were added:

**Skills registry sync** — reads every skill file and compares frontmatter against the skills index table. Adds missing rows, updates changed schedules. The index was drifting from reality because nobody was updating it consistently.

**Vault index update** — reads the top-level project structure and updates the vault's navigation index. Adds new projects, updates stale descriptions, adjusts the last-updated timestamp.

**Monthly digest** — on the first Sunday of each month only, reads all daily notes from the previous month, extracts open tasks, ideas, and learning items, and writes a single digest file. The condition is `DAY_OF_MONTH -le 7`. On other Sundays, this step is skipped entirely.

One fix was required during this build: the original sleep gap between the finance check-in and the weekly review was 30 seconds. That was not enough. The finance check-in reads seven daily notes and writes a structured section — it takes three to five minutes to complete. A 30-second gap meant the weekly review was starting before finance had finished.

The gap was changed to 300 seconds. This is the correct framing: when chaining Claude sessions, the sleep gap must be at least as long as the expected runtime of the preceding session, not just a polite pause.

### Morning Briefing: Golden Hour Check

The existing morning briefing already surfaced the LinkedIn queue for the day. A gap remained: if a post had gone live overnight or in the early morning, the briefing had no way to flag that the first hour of comments was the highest-leverage engagement window.

LinkedIn's algorithm weights early engagement heavily. Responding to comments within the first 60 minutes after posting consistently increases reach. If I posted at 8:00 AM and opened my vault at 8:30 AM, I needed the briefing to tell me that, not rely on me to remember it.

The check added to the briefing reads the performance log for any post published within the last 48 hours. If a post is less than 24 hours old, the briefing adds a single line:

```
**Golden Hour:** reply to comments on "[post title]" — posted [X] hours ago
```

If no post is within that window, the line is omitted entirely. No noise when there is nothing to act on.

### Code Review Pre-Push Hook

Every project accumulates technical debt through small decisions made under time pressure. A code review step that runs only when remembered is a code review step that runs rarely.

A pre-push git hook in the support dashboard repository calls `claude -p` with the diff of commits about to be pushed. The review checks for security issues at system boundaries, TypeScript gaps, Supabase query patterns, and debug code left in. The output is appended to a log file inside the `.git/` directory.

The hook is non-blocking — it always exits 0. The purpose is information, not gatekeeping. Issues are surfaced. The decision to fix before pushing belongs to the developer.

### Publish Session: QA Gate Before Push

The skill that converts vault session notes to blog posts had no quality check before the commit. Content rules that apply to published writing — no fabricated numbers, no internal shorthand, no vault-specific references — were checked manually only if I remembered to check.

A new step was added before the push: the agent reads the rewritten post and checks it against the rules before committing. Fabricated numbers are removed. Contractions and abbreviations are expanded. Internal vault paths and persona names are caught. Fixes are made inline without prompting. The commit happens only after the check passes.

## The Architecture Pattern

Every runner in this system uses the same structure:

1. A shell script that validates prerequisites (vault exists, Chrome running if needed), calls `claude -p` with an explicit prompt and tool allowlist, and logs the result
2. A LaunchAgent plist that schedules the shell script using `StartCalendarInterval`
3. Log rotation: logs older than 14 days are deleted at the top of every run

The `StartCalendarInterval` detail matters. macOS cron skips jobs when the machine is asleep. `StartCalendarInterval` fires on the next wake. For daily vault maintenance, "fire on wake" is the correct behaviour — a briefing that ran at 11:47 AM because the Mac was asleep at 8:30 AM is still useful. A briefing that was silently skipped is not.

## What the System Looks Like Now

Seven active runners, all on the same architecture:

| What | When |
|---|---|
| Morning briefing | 8:30 AM IST daily |
| LinkedIn analytics sync | 9:07 AM IST daily (Chrome guard) |
| Inbox processor | 9:30 PM IST daily |
| Done-tasks migrator | 10:00 PM IST daily |
| Draft post dispatcher | Monday 10:00 AM IST |
| Sunday bundle (finance + review + maintenance) | Sunday 6:00 PM IST |
| Monthly digest (inside Sunday bundle) | First Sunday of month only |

Two hooks in the development workflow:

- Pre-push code review on the support dashboard repository
- QA gate inside the publish-session skill before every blog commit

## What I Learned

**The sleep gap between chained sessions must match actual runtime, not feel right.** `sleep 30` between two Claude sessions looks reasonable. A Claude session that reads seven files and writes structured output takes three to five minutes. The gap needs to match the real number.

**Guard clauses on environment dependencies prevent silent failures.** An agent that requires Chrome to be open will fail silently if Chrome is not there. A two-line process check at the start of the script converts a silent failure into a clean skip with a log entry. The distinction matters when you are trying to diagnose why a job did not run.

**Non-blocking quality gates are better than no quality gates.** A code review hook that blocks pushes gets disabled within a week. A hook that surfaces issues and exits cleanly stays on permanently and provides value every time it runs.

**Monthly operations belong inside weekly runners, not separate jobs.** The monthly digest runs as a conditional step inside the Sunday bundle rather than a separate LaunchAgent. Fewer moving parts. The condition (`DAY_OF_MONTH -le 7`) is the correct primitive for "first Sunday of the month" on macOS.

**Automation maintenance is now automated.** The skills registry sync and vault index update run weekly without manual effort. The system that maintains the system is now part of the system.

---

**[2026-04-15](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![automation](https://img.shields.io/badge/automation-2ea44f?style=flat-square&logoColor=white)](../README.md#all-posts) [![macos](https://img.shields.io/badge/macos-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-2ea44f?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the first three LaunchAgents this post expands on; covers the CronCreate failure and why LaunchAgents were the right architecture
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — the silent failure that affected the prior generation of this system; directly informs the guard clause pattern used throughout this post
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — the same quality-gate thinking applied to the publish-session QA step; a producing agent cannot eval its own output
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the persona coordination layer that these automated routines run on top of

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

<!-- source_session: 2026-04-05_vault-os-complete -->

# From Manual to Automatic: How I Built a Vault OS That Runs Itself

> I stopped using AI and started letting it run for me. Here is exactly how.

## Context

For years, the promise of a personal knowledge system was: write everything down, and you will have access to it later. The reality was: write everything down, forget where it is, and spend 10 minutes at the start of every working session reconstructing what you were doing last time.

I use Obsidian as my vault and Claude Code as my AI coding and thinking partner. By April 2026 I had built up a real system, but it still required me to manually trigger every AI workflow. I had to remember to run the morning briefing. I had to remember to process my daily note captures. I had to remember to do the weekly review.

That is not a system. That is a checklist that depends on me.

This post documents how I replaced that with three automated agents that run on schedule, without any manual triggering, and leave outputs in my vault where I can read them the next morning.

## The Problem with CronCreate

My first attempt at scheduling used Claude Code's built-in `CronCreate` tool with `durable: true`. The parameter exists. The documentation suggests it persists to disk. It does not.

After a session restart, every scheduled job was gone. The `durable` flag was silently unreliable.

Once I stopped trying to fix that and looked for the right tool instead, the answer was obvious: macOS LaunchAgents. They are the native macOS primitive for scheduled background jobs. They persist across reboots. They have proper logging. They are what system services use.

## The Three Agents

I built three shell scripts and three LaunchAgent plist files:

**Morning Briefing** (fires at 8:00 AM IST daily)

Reads the vault's `active-context.md`, surfaces open tasks, flags anything overdue, and appends a brief to the current day's note. On Mondays it runs a holistic review instead of the standard brief.

**Inbox Processor** (fires at 9:00 PM IST daily)

Reads the daily note, routes tagged captures to their destinations. A line tagged `#idea` goes to the ideas backlog. A line tagged `#learn` goes to the reading queue. A line tagged `#linkedin` gets queued for the content pipeline. No manual sorting required.

**Weekly Review** (fires at 7:00 PM IST on Sundays)

Reads all sessions from the week, all open tasks across projects, LinkedIn post performance, and produces a structured weekly summary. It also runs a lint pass on active projects to flag stale claims and broken paths.

## How LaunchAgents Work

Each agent is two files:

1. A shell script that calls `claude -p` with an explicit tool allowlist:

```bash
claude -p "run the morning briefing skill" \
  --allowedTools Read,Write,Edit,Bash \
  --add-dir "/Users/aw/Library/Mobile Documents/com~apple~CloudDocs/Obsidian/Aw Vault"
```

2. A plist file in `~/Library/LaunchAgents/` that tells macOS when to run it:

```xml
<key>StartCalendarInterval</key>
<dict>
  <key>Hour</key>
  <integer>2</integer>
  <key>Minute</key>
  <integer>30</integer>
</dict>
```

One important detail: `StartCalendarInterval` uses UTC, not local time. IST is UTC+5:30 (not a round number), so the conversions need to be explicit. 8:00 AM IST is 2:30 AM UTC. Getting this wrong means your morning briefing fires at 2:00 PM.

The tool allowlist matters too. I restrict each agent to only `Read`, `Write`, `Edit`, and `Bash`. Vault automation does not need access to web tools or anything outside the vault directory. Restricting scope prevents automated jobs from doing things they were not designed to do.

## The First Successful Run

The smoke test on April 5 confirmed the system was working. I ran the morning briefing script manually and it completed in under two minutes. The output was a Morning Brief appended to that day's daily note:

- ChalotripBot had 3 open issues flagged
- 20 personal tasks were listed
- A GitHub reading queue item was flagged as 10 days old with no action taken

That last item mattered. I had forgotten about it entirely. The system remembered.

## What Actually Changed

The shift is not about saving time in a measurable sense. It is about removing the activation cost of starting. When I open my laptop in the morning, the brief is already there. I do not have to run anything. I do not have to remember to run anything.

The system has been running every day since April 5 without manual intervention. The logs confirm successful runs. The vault confirms the outputs are landing correctly.

This is what a knowledge system should feel like: ambient, not demanding.

## What I Learned

**CronCreate is for in-session reminders, not persistent automation.** Use macOS LaunchAgents or system crontab for anything that needs to survive a session restart.

**Always restrict tool scope in automated jobs.** An automated agent with unrestricted tool access is a liability. List exactly what it needs and nothing more.

**UTC conversion for IST requires care.** IST is UTC+5:30. Always document the conversion explicitly in the plist file or the shell script header. A missed 30 minutes will shift your scheduled run by half an hour every day.

**Verify actual state after an agent runs.** The completion notification tells you the job finished. It does not tell you the job succeeded. Always read the output.

---

**[2026-04-05](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![obsidian](https://img.shields.io/badge/obsidian-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![automation](https://img.shields.io/badge/automation-2ea44f?style=flat-square&logoColor=white)](../README.md#all-posts) [![launchagent](https://img.shields.io/badge/launchagent-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![knowledge-management](https://img.shields.io/badge/knowledge--management-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [Three Claude Tools, One Vault](three-claude-tools-one-vault.md) — the shared context architecture this automation runs on top of
- [How I Retired Notion in One Session](how-i-retired-notion.md) — consolidating the knowledge base that feeds these automated agents
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the next step after automating individual workflows was building a coordinated team of AI personas that share knowledge
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the v3 agent architecture that runs inside this system
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — what broke in the scheduling layer of this exact system, how it was diagnosed, and why "registered" is not "running"

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

<!-- source_session: 2026-04-23_vault-iq-tier1 -->

# AI Runners That Remember

*2026-04-23 · automation, ai-agents, claude-code, memory, vault-os · Vault as OS*

Most AI automation pipelines have amnesia.

A cron job fires. Claude reads some files. Claude writes some output. The job ends. Tomorrow, the same job fires again with zero memory of what happened yesterday. It re-evaluates everything from scratch. It makes the same conservative calls it made last week. It cannot tell you whether a pattern is gaining momentum or quietly dying.

I fixed this. Here is what I built, why it works, and how you can implement it.

---

## The problem: stateless runners

I have six scheduled AI runners firing on my personal knowledge vault. Morning briefing, pattern extraction, weekly review, LinkedIn analytics, and more. Each one uses `claude -p` (Claude Code CLI) with `--allowedTools Read,Write,Edit,Bash`.

They work. But every run starts cold.

The pattern extractor runs every Sunday. It reads sessions from the past week and decides which cross-project patterns are strong enough to keep in `patterns.md`. But it cannot compare this Sunday to last Sunday. It cannot notice that "Silent job failure" has appeared in 4 consecutive sessions and is deeply embedded, while "Bidirectional cross-linking" has had zero references for 14 days and should probably be archived.

That is a real loss of signal.

---

## What I researched

Before building anything, I verified the claims:

**Anthropic's memory tool + context editing**: On an internal agentic search evaluation, combining memory with context editing improved task quality by 39% and reduced tokens 84% on 100-turn workflows. Source: [anthropic.com/news/context-management](https://www.anthropic.com/news/context-management). Exact match — no embellishment.

**Letta's sleep-time compute**: Pre-computing context during idle periods reduces test-time compute by approximately 5x on stateful reasoning benchmarks. Source: arxiv 2504.13171. The insight: AI agents that "think while sleeping" produce cleaner, more actionable memory for active sessions.

**Anthropic's context engineering article**: Their team describes "structured note-taking" as the foundational pattern — agents writing notes to files and reading them back. Quote: *"like Claude Code creating a to-do list... this simple pattern allows the agent to track progress across complex tasks."* This validated the approach before I wrote a line.

One important correction from the research: Anthropic's official memory tool (`memory_20250818`) is API-only — it requires a beta header and client-side CRUD handlers. It is not a CLI flag. For `claude -p` runners, the equivalent is instructing Claude to read and write markdown files using its existing Read/Write/Edit tools. Same effect, no SDK rewrite.

---

## What I built: three pieces

### 1. Runner memory store

A directory: `Knowledge/memory-store/` with one `.md` file per runner.

Each file has five sections:

```markdown
## Strategy for next run
[Adaptive instructions from last run — Claude reads and acts on these immediately]

## Self-assessment
Quality: HIGH/MEDIUM/LOW
Confidence: 85%
Issues: none

## [Runner-specific observations]
...

## Watch next time
...
```

The `## Strategy for next run` section is the key innovation. It is not just notes — it is behavioral instructions written by the previous run for the current run. The pattern extractor, after its Sunday run, writes things like:

> "Linkwhisper had most raw material this week — start there next Sunday. Task-Automation project has had zero sessions for 3 consecutive weeks — flag as dormant if still empty. Do not re-promote 'Enhance in-place' without new cross-project evidence."

Next Sunday, the runner reads this and acts on it before touching any other file. No human prompt needed.

Each runner now has two new prompt sections:

**MEMORY LOAD** (at the start):
```
Read Knowledge/memory-store/[runner]-memory.md.
Read '## Strategy for next run' carefully — these are adaptive instructions from your prior run.
Act on them immediately as you execute the skill.
```

**MEMORY WRITE** (at the end):
```
Overwrite Knowledge/memory-store/[runner]-memory.md with your observations.
Write specific, named instructions in '## Strategy for next run'.
Rate your output quality in '## Self-assessment'.
Keep under 300 words. Overwrite completely.
```

This is the structured note-taking pattern from Anthropic's context engineering article, applied to scheduled automation.

### 2. Spaced repetition queue

Decisions and past mistakes are only useful if you see them at decision time.

I created `Knowledge/srs-queue.md` — a simple markdown table with 8 seeded items:

```markdown
| Item | Source | Last seen | Next due | Interval (days) |
|---|---|---|---|---|
| launchd scripts must live in ~/.claude/vault-runners/ — never inside iCloud folders | past-mistakes.md | 2026-04-23 | 2026-04-24 | 1 |
| sleep gaps between chained runners must be ≥ expected runtime of prior step | past-mistakes.md | 2026-04-23 | 2026-04-24 | 1 |
...
```

The morning briefing skill now has a Step 0.5: read this file, find rows where `Next due ≤ today`, surface the first three under `## Remember today` at the top of the brief.

Today's actual brief surfaced:
```
## Remember today
- launchd scripts must live in ~/.claude/vault-runners/ — never inside iCloud folders
- sleep gaps between chained runners must be ≥ expected runtime of prior step  
- node not on PATH in launchd — set PATH explicitly at top of every new runner script
```

That day's Top 3 included building new LaunchAgent plists. The SRS section surfaced the three most costly runner mistakes at exactly the moment they mattered.

The weekly review runner advances intervals each Sunday using simplified SM-2: `interval × 2`, capped at 30 days. Items not seen reset to overdue and keep resurfacing until acknowledged.

No Obsidian plugin required. All existing SRS plugins require flashcard `Question::Answer` format — incompatible with decision block structure. A standalone table is cleaner and fully automatable.

### 3. Sleeptime consolidator

A new runner fires at 2:00 AM daily. It uses `--model claude-opus-4-7` (stronger model for higher-quality consolidation).

What it does:

1. Reads all session files from the last 7 days
2. Counts how many sessions referenced each Active pattern in `patterns.md`
3. Adds strength signals: `⟦HIGH | 4 refs this week⟧`, `⟦LOW | ⚠️ 0 refs this week⟧`, `⟦⚠️ FADING — 0 refs 14+ days⟧`
4. Detects overlaps between `patterns.md` and `past-mistakes.md` (same lesson described twice) and annotates them
5. Flags archive and promote candidates for human review — never auto-archives
6. Writes a consolidation report to `Knowledge/sessions/`

Before sleeptime:
```
## Silent scheduled-job failure needs structured observability
- Where learned: Linkwhisper (Apr 14), Vault automation (Apr 14), PMDojo (Apr 18)
...
```

After first run:
```
## Silent scheduled-job failure needs structured observability ⟦HIGH | 4 refs this week⟧
- Where learned: Linkwhisper (Apr 14), Vault automation (Apr 14), PMDojo (Apr 18)
_Specific instances: "A loaded plist is not proof the agent can run", "Vercel cron handlers must export GET + Authorization Bearer"_
...
```

The strength signal tells every session tomorrow: this lesson is deeply embedded and actively relevant. The overlap annotation merges two separate memory entries without deleting either.

This is the sleep-time compute concept applied to a personal knowledge system: the model reorganizes what it learned while idle, so active sessions start with higher-quality memory.

---

## How to implement this yourself

You need: Claude Code CLI, scheduled runners using `claude -p`, and a file system your runners can read/write.

**Step 1: Create the memory store**

```bash
mkdir -p your-project/memory-store
```

For each runner, create a stub:
```markdown
# [Runner Name] Memory
_First run — no prior observations._

## Strategy for next run
[Filled after first run]

## Self-assessment
Quality: UNKNOWN (first run)

## Observations
[Filled after first run]

## Watch next time
[Filled after first run]
```

**Step 2: Add MEMORY LOAD to each runner's prompt** (at the start):

```
MEMORY LOAD: First, read memory-store/[runner]-memory.md.
Read '## Strategy for next run' — these are adaptive instructions from your prior run.
Act on them immediately. If quality was LOW in '## Self-assessment', adjust your approach.
```

**Step 3: Add MEMORY WRITE to each runner's prompt** (at the end):

```
MEMORY WRITE: Overwrite memory-store/[runner]-memory.md.
Write specific named adaptive instructions in '## Strategy for next run'.
Rate your output quality in '## Self-assessment'.
Keep under 300 words. Overwrite completely.
```

**Step 4: Create a review table** (for SRS, optional):

```markdown
| Item | Source | Last seen | Next due | Interval (days) |
|---|---|---|---|---|
| [your most important past mistake] | mistakes.md | [today] | [today+1] | 1 |
```

Surface items where `Next due ≤ today` in your daily brief. Advance intervals weekly.

**Step 5: Add a sleeptime runner** (optional, high value):

```bash
claude -p "Read session files from the last 7 days. For each pattern in patterns.md, count references. Add strength signals. Flag fading patterns. Write a consolidation report." \
  --model claude-opus-4-7 \
  --allowedTools "Read,Write,Edit,Bash,Glob,Grep"
```

Schedule at 2am. Use a stronger model. The run takes 3–5 minutes. The benefit compounds daily.

---

## What this actually changes

Before Tier 1:
- Runners re-evaluate everything from scratch every run
- Important lessons live in files that are read only at boot
- Patterns can fade to noise without anyone noticing

After Tier 1:
- Each runner's next run is guided by the previous run's strategy
- SRS surfaces the highest-value lessons at exactly the moment they're relevant
- Patterns.md has live strength signals — stale patterns are visible before they cause drift
- The system gets smarter by running

**On token savings:** these are runner-specific, not universal. The daily briefing runner adds a small amount of input context (memory file + SRS queue) but gains strategic guidance. The pattern-extractor runner saves meaningfully — the sleeptime consolidator pre-computes reference counts overnight, so the Sunday run no longer needs to scan seven days of session files to derive pattern strength. That scan is ~7,000 words replaced by ~20 words of pre-computed velocity data. Quality improvement is consistent across all runners. Token reduction is significant for the pattern-extractor and sleeptime consolidator specifically.

The research claims hold: structured note-taking produces coherence across sessions. Sleep-time compute produces cleaner memory for active sessions. Both are achievable with file I/O and existing tools — no new API, no SDK rewrite.

---

## Does it work? The eval results

Before shipping, I ran a structured evaluation against the system. Here are the real numbers.

**Token measurement (actual file sizes):**

| Context | Words | ~Tokens | Role |
|---|---|---|---|
| past-mistakes.md Active | 2,968 | ~3,948 | Pattern-extractor scans this in full |
| patterns.md Active | 780 | ~1,037 | All runners read this |
| 7 days of session files | ~7,000+ | ~9,300+ | Pattern-extractor counted refs manually — before |
| Memory file (per runner) | 76–300 | ~100–400 | Replaces from-scratch analysis |
| SRS queue | 290 | ~386 | Surfaces 3 items, skips 2,968 words |

**Where token savings actually land:**
Morning-briefing adds ~366 words of input (memory file + SRS queue) and gains strategic guidance. This is a quality trade, not a token trade.

Pattern-extractor is where the numbers matter: the sleeptime consolidator pre-computes reference counts overnight. The Sunday runner no longer scans 7 days of session files to count how often each pattern appeared. That ~7,000-word scan is replaced by ~20 words of pre-computed velocity data in the memory file.

**First real run — evidence:**

The morning-briefing ran its first automated run with live memory on 2026-04-23 at 12:56 IST. The memory file it wrote back:

```
## Strategy for next run
Vault IQ Tier 1 is DONE (completed Apr 23) — Board still shows it in Active.
Override the board and pull from Up Next.
Beehiiv email capture has been in Top 3 for 6 consecutive mornings — add urgency:
"6 mornings, still not started — commit or kill."

## Self-assessment
Quality: HIGH
Confidence: 82%
Issues: Board lagged behind daily note — had to cross-reference work log to get
accurate Top 3. Future runs: if daily note has [x] completed for a Board Active
card, treat it as done.
```

That is a behavioral instruction, not an observation. The next run reads "override the board" and acts on it — without being told by a human. The self-assessment surfaces a real issue the runner found in its own execution. That is what the system was designed to produce.

**Specificity gate in practice:**

Before the gate, the Strategy section could contain "Continue monitoring projects" — which satisfies the format but provides no guidance. The gate added to every MEMORY WRITE prompt: *"If you cannot name a specific project, file, pattern, or date in this instruction, it is too generic to include."*

The first real run produced named instructions: "Vault IQ Tier 1," "Board," "Beehiiv email capture," "6 consecutive mornings," "Apr 28 Mon slot." Zero generic lines.

**Catch-up system (edge case coverage):**

The three-layer catch-up handles the most common failure mode — scheduled runner fires when laptop is asleep:

1. Done-file + lock in every runner: idempotent, safe to run from multiple triggers
2. Catch-up LaunchAgent (`StartInterval: 1800`): checks every 30 min, fires within 30 min of laptop wake
3. Session-loader hook: fires instantly when any Claude session opens, surfaces notice and background-triggers the runner

The catch-up plist uses `RunAtLoad: true` — it fires the moment it is loaded. On first load today it detected the morning briefing as missed (run was from before done-files existed), triggered it, and received exit 0 in under 5 minutes. The automation-health log shows the catch-up entry.

---

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the foundational automation layer this memory system runs on top of; if your runners are not yet scheduled, start here
- [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](second-automation-layer.md) — the expansion layer that brought the runner count to seven; covers the sleep-gap bug that the SRS queue now explicitly resurfaces at the right moment
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — the session immediately before this one: adding Reflexion Blocks and a user profile; the memory store here is the next step in that same compounding arc
- [My AI Agents Were Running. They Just Were Not Working.](self-correcting-agents.md) — what happens when agents run but do not self-correct; the memory store solves the same problem at the runner level
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — the silent failure that the SRS queue now resurfaces every time new LaunchAgent plists are built
- [I Cherry-Picked a Viral Cost-Cut Post](cherry-picking-the-cost-post.md) — the measurement loop this runner system will host: weekly quota burn rate compared against a baseline, the same way it currently surfaces past-mistakes velocity

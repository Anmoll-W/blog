<!-- source_session: 2026-04-23_vault-iq-tier1, 2026-06-25_ecc-observation-synthesis -->

# The Write-Only Trap

*2026-06-25 · automation, ai-agents, claude-code, memory, vault-os · Vault as OS*

I had a PostToolUse hook that captured every tool call my AI agents made. Every file they read, every command they ran, every edit they applied — timestamped, structured JSON, appended in real time.

Nobody was reading it.

That is not an observation system. That is a very organized filing cabinet.

---

## The two gaps that look like one

The obvious problem with scheduled AI runners is amnesia. A cron job fires. Claude reads some files. Claude writes output. The job ends. Tomorrow the same job fires with zero memory of what happened yesterday. It re-evaluates everything from scratch. It makes the same conservative calls it made last week.

The less obvious problem is that even when you add memory, the system still cannot tell you what behavioral patterns are actually emerging across sessions. It knows what it wrote to its own memory file. It does not know what it keeps doing without noticing.

Both gaps have the same root: data flows one way. Runners write observations that nothing reads.

I built three pieces to close these loops. The first two are about giving runners persistent cross-run memory. The third is about closing the write-only trap — the gap that only becomes visible when you audit the whole system from outside it.

---

## The first piece: runner memory

I run scheduled agents on my personal knowledge vault. Morning briefing, pattern extraction, weekly review, and more. Each uses `claude -p` with `--allowedTools Read,Write,Edit,Bash`.

They work. Every run starts cold.

The pattern extractor runs every Sunday. It decides which cross-project patterns are strong enough to keep in `patterns.md`. But it cannot compare this Sunday to last Sunday. It cannot notice that "silent job failure" appeared in four consecutive sessions and is deeply embedded, while "bidirectional cross-linking" has had zero references for 14 days and should probably be archived.

The fix is a memory store: a directory called `Knowledge/memory-store/` with one `.md` file per runner. Each file has four sections. The key one is `## Strategy for next run`.

This is not notes. It is behavioral instructions written by the previous run for the current run. The pattern extractor, after its Sunday run, writes:

> "Linkwhisper had the most raw material this week — start there next Sunday. Task-Automation project has had zero sessions for three consecutive weeks — flag as dormant if still empty. Do not re-promote 'Enhance in-place' without new cross-project evidence."

Next Sunday, the runner reads this before touching any other file and acts on it. No human prompt needed.

Each runner gets two new prompt sections. A MEMORY LOAD at the start:

```
Read Knowledge/memory-store/[runner]-memory.md.
Read '## Strategy for next run' — these are adaptive instructions from your prior run.
Act on them immediately.
```

And a MEMORY WRITE at the end:

```
Overwrite Knowledge/memory-store/[runner]-memory.md.
Write specific, named instructions in '## Strategy for next run'.
Rate your output quality in '## Self-assessment'.
Keep under 300 words. Overwrite completely.
```

Anthropic's context engineering research describes this as "structured note-taking" — the foundational pattern for agents that need to track progress across complex tasks. Applied here: the agent writes notes to a file and reads them back on the next run.

The first real run produced this in the memory file:

```
## Strategy for next run
Vault IQ Tier 1 is DONE (completed Apr 23) — Board still shows it in Active.
Override the board and pull from Up Next.
Beehiiv email capture has been in Top 3 for six consecutive mornings — add urgency:
"six mornings, still not started — commit or kill."

## Self-assessment
Quality: HIGH
Confidence: 82%
Issues: Board lagged behind daily note — had to cross-reference work log to get
accurate Top 3. Future runs: if daily note has [x] completed for a Board Active
card, treat it as done.
```

That is a behavioral instruction, not an observation. The next run acts on "override the board" without being told by a human.

---

## The second piece: spaced repetition

Past mistakes are only useful if you encounter them at decision time.

I created `Knowledge/srs-queue.md` — a markdown table with past mistakes as rows. Each row has a `Next due` column. The morning briefing skill reads this file at the start of every run and surfaces the first three items where `Next due ≤ today`.

One actual brief:

```
## Remember today
- launchd scripts must live in ~/.claude/vault-runners/ — never inside iCloud folders
- sleep gaps between chained runners must be ≥ expected runtime of prior step
- node not on PATH in launchd — set PATH explicitly at top of every new runner script
```

That day, I was building new LaunchAgent plists. All three of those mistakes were live risks. The SRS section surfaced them before any code was written.

The weekly review runner advances intervals each Sunday using simplified SM-2: interval multiplied by two, capped at 30 days. Items past their due date resurface until acknowledged.

No Obsidian plugin required. A standalone table is cleaner and fully automatable.

---

## The third piece: closing the write-only trap

Here is what I had but did not realize I had: a PostToolUse hook that captured every tool-use event from every Claude Code session to a structured JSONL file. Every Read, Write, Edit, Bash call — `{"ts":"...","tool":"Edit","target":"/path/to/file","status":"unknown"}`.

I reviewed the hook output and saw clean, structured JSON. I did not immediately ask: where does this go? That is a question you rarely think to ask when the capture is working.

The hook was writing. Nothing was reading. The file was growing daily, and the agents had no access to it, no awareness of it, and no way to learn from it.

When I audited the system against a framework for agent observability, the gap surfaced immediately under one heading: *consumer absent*. A producer with no consumer is not an observation system. It is a write-only store that happens to contain signal.

The consumer I built is called the observation synthesis runner. It runs every Sunday, after the pattern extractor, before the skills README sync. Here is what it does:

**Step 0 — Find files:** Locate all JSONL files in `Knowledge/automation-data/` that are newer than the last watermark. If none exist, skip cleanly.

**Step 1 — Load and filter:** Parse each line as JSON. Drop any line where both `tool` and `target` are unknown — that is noise, not signal. Group events by calendar day. Count: for each `(tool, target)` pair, how many distinct sessions did it appear in?

**Step 2 — Threshold:** Only pairs present in two or more distinct sessions are candidates. A single-session event is not a pattern.

**Step 3 — Match against the corpus:** Compare candidates against `shared-mistakes.md`, `shared-patterns.md`, and `decisions.md`. Classify each as CONFIRMS (already documented — skip), EXTENDS (new frequency or context for a known pattern — candidate), CONTRADICTS (behavior conflicts with a known rule — high priority candidate), or NEW (no match — candidate).

**Step 4 — Write candidates:** Append up to five candidates to `## Proposed Promotions` in `shared-patterns.md`. Every candidate is verbatim from the JSONL — tool name, target path, session count. Nothing is inferred or paraphrased.

**Step 5 — Health log and watermark:** Append a summary to `automation-health.md`. Advance the watermark so next week only processes new files.

The reliability pattern mirrors every other runner in the system: a done file prevents double-runs on the same day, a lock file with a dead-PID check handles stale processes, the watermark prevents re-processing files that were already consumed. Five stress tests on the first day — idempotency, stale lock cleanup, watermark filter, concurrent lock, JSONL parse validity — all passed.

The first real run processed 87 events from one calendar day and found zero candidates. That is correct. Zero is information. The threshold requires two or more distinct sessions, and there was only one session — the day I built it. The system will surface real patterns as sessions accumulate.

---

## The Hermes extension

The runner runs locally on the Mac every Sunday. But the Mac is off half the time.

I added the same job to Hermes — a small AI gateway running on Railway. The job reads the same vault JSONL files (synced via GitHub), runs the same synthesis logic, and writes candidates back to the same `shared-patterns.md` file. The vault git sync runner picks up the write on its next 15-minute tick.

The Sunday runner fires whether or not the laptop is on.

---

## How the three pieces interlock

The memory store closes the run-to-run amnesia gap. Each runner carries its own strategy forward.

The SRS queue closes the session-to-session mistake recurrence gap. Lessons surface when they are relevant, not when they are written.

The observation synthesis closes the behavioral blind spot gap. Patterns that emerge from what the agents actually do — not what they write about — get surfaced for human review.

None of these pieces talk to each other directly. They share a vault. The vault is the coordination layer.

---

## How to implement this

You need: Claude Code CLI (`claude -p`), scheduled runners, and a file system your runners can read and write.

**Runner memory:**

Create a stub memory file per runner:
```markdown
# [Runner Name] Memory
_First run — no prior observations._

## Strategy for next run
[Filled after first run]

## Self-assessment
Quality: UNKNOWN
```

Add MEMORY LOAD at the start of each runner prompt. Add MEMORY WRITE at the end. The specificity gate matters: if the strategy section says "continue monitoring" — that is too generic. It should say a project name, a file, and a date.

**SRS queue:**

Create `srs-queue.md` with a row per past mistake and columns for last seen, next due, and interval in days. Have your morning briefing runner surface rows where next due is today or earlier. Have your weekly review runner advance the intervals using interval multiplied by two.

**Observation synthesis:**

Wire a PostToolUse hook that appends JSON to a JSONL file on every tool call. Build the consumer runner that reads those files weekly, groups by session, applies the two-session threshold, and writes classified candidates to a shared file under a `## Proposed Promotions` header. Never auto-approve — only surface.

---

## What changes

Before this system:
- Runners restart from scratch on every run
- Past mistakes live in a file that is read at boot and forgotten by session two
- Tool-use patterns are invisible — the system has no access to its own behavioral data

After:
- Each runner carries its prior run's strategy as a behavioral instruction
- Past mistakes resurface at exactly the moment they are relevant, not just at boot
- Behavioral patterns across sessions are captured, classified, and surfaced for review

The product decision underneath all three pieces is the same one: data that flows only in one direction is not a feedback loop. A write-only store is not memory. Memory requires a reader that acts on what was written.

---

## Related

- [What I Learned Auditing an AI Agent Repository Built by Another Product Manager](auditing-another-pms-agent-repo.md) — the audit session that surfaced the write-only trap explicitly; the observation synthesis runner in this post is the direct response to that audit's top finding
- [Every Status Was Green. Three of Them Were Lying.](every-status-was-green.md) — the monitoring layer that exposed silent failures on its first day; the proof-carrying-signal pattern here applies directly to the watermark and health log in the synthesis runner
- [My AI Agents Were Running. They Just Were Not Working.](self-correcting-agents.md) — what happens when agents run reliably but produce no self-correction; the memory store here is the next structural step after that audit
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the foundational automation layer all three pieces in this post run on top of
- [Why I Shut Down Hermes — a Multi-Agent AI System I Built Myself](why-i-shut-down-hermes.md) — the shutdown post that cites the observation synthesis runner here as the simpler thing that actually runs; the reflector loop Hermes never shipped is the same problem this post solves locally

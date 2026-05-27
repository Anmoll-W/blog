<!-- source_session: 2026-05-18_token-burn-audit -->

# I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.

*2026-05-18 · claude-code, automation, token-efficiency, silent-failures, launchagent · Silent Failures*

I had done the token optimization work. Cache discipline. `/effort` per prompt. Subagent model routing. Four levers adopted from a viral cost-cut post after running each one through a verification framework.

The quota was still burning faster than it should.

I ran an audit. What I found was not a configuration problem. It was five separate automation bugs, each invisible on its own, each burning sessions in the background while the optimizations I had applied were trying to hold the line.

---

## Bug 1: 18 sessions burned in one day instead of 1 per week

A `vault-catchup` LaunchAgent fires every 30 minutes. Its job is to catch any scheduled weekly job that missed its Sunday window — network down, system asleep, whatever. The catchup checks each runner's done file and fires the job if the done file is missing or stale.

The `linkedin-analyst` runner is supposed to run once per Sunday.

Last Sunday's run timed out and exited with `exit 1`.

The catchup only accepted `exit 0` as a successful completion. Exit 1 meant no done file was written. So from Monday onward, every 30-minute catchup interval found the analyst's done file missing, decided the job hadn't run, and spawned a full Claude session to run it.

By the time I audited: 18 Claude sessions fired that day for a job that should run once per week.

The fix was two things. First, write the done file regardless of exit code — the done file tracks *attempted*, not *succeeded*. Second, change the catchup to scan the full Mon-Sun window rather than just checking whether a done file exists for the current run.

Done-file contracts need to be explicit: are they tracking completion or attempt? The catchup assumed completion. The runner assumed completion. When the runner failed, both assumptions broke simultaneously and neither one made noise about it.

---

## Bug 2: A 7-hour Claude session every night

The `sleeptime-consolidator` runs at 2:00 AM to compress memory, annotate patterns, and write behavioral handoffs for the next day's runners. It used a Perl one-liner as a timeout guard:

```bash
perl -e 'alarm(900); exec @ARGV' -- claude -p "run consolidator" ...
```

The intent: 15-minute alarm, then kill the process.

The problem: on macOS, `alarm()` is reset to 0 when `exec` replaces the Perl process. The Perl process disappears and the timer goes with it. There is nothing left to fire the alarm. Claude runs until the OS kills it.

Logs showed sessions from 2:00 AM to 9:11 AM. Seven hours and eleven minutes, every night.

The fix was `timeout 1800 claude -p ...` — a real external watchdog that survives exec. The `timeout` command wraps the child process; it does not become it.

If you are using `perl -e 'alarm(N); exec @ARGV'` as a timeout pattern anywhere, it does not work on macOS. The exec replaces the process, the alarm timer is not inherited, and you have no timeout at all.

---

## Bug 3 and Bug 4: Two zombie LaunchAgents

A post-deploy watcher was designed to self-disable after 7 firings. It had logged 8 firings. The self-disable script was called. It failed silently — the `launchctl unload` ran before the plist was where `launchctl` expected it. The agent stayed loaded, kept firing daily, kept spawning Claude sessions for a watcher job that had already done its work.

A one-shot test job from May 11 had the same profile: self-disable logic that ran after an `exec`, meaning it ran in a process that had already been replaced. The disable never happened. The job kept firing.

Neither of these showed up in any error log. `launchctl list` showed both as loaded and running. From the outside they looked exactly like healthy agents.

The pattern in both: self-disable logic that depended on the job's own process being alive at the time of disable. That assumption does not hold when the disable step comes after an `exec` or depends on a path that changes between registration and fire time.

---

## Bug 5: 180,000 characters loaded on every session start

The vault's BOOT sequence loads context at the start of every interactive session. The instruction was to Read() three files: `patterns.md`, `decisions.md`, and `anmoll-profile.md`.

`decisions.md` had 81 entries added since April. The Active section alone was 103,435 characters. `patterns.md` was 45,000 characters. Together with the profile, BOOT was loading roughly 180,000 characters of context on every first message of every session.

None of that content was designed to be read in full on every session. Most of it was history that was relevant once and never needed again.

The fix was two-part. Compressed 81 old decision entries down to 1-line summaries — from 103,435 characters to roughly 34,000. Then updated BOOT to use a pre-loaded session-loader file with a 3,500-character cap instead of full file reads. The loader contains the current-state summary; the full files exist if Claude needs to dig in.

The savings compound: every interactive session now starts with a 3,500-character context load instead of a 180,000-character one.

---

## Bug 6: autoCompact was never enabled

Long sessions accumulated context with no compaction. The `autoCompactEnabled` flag in `settings.json` was false — the default. It had never been set.

This one is short because the fix is short: set `autoCompactEnabled: true`. But it is worth naming because it is the kind of silent accumulation that never throws an error. Sessions just get slower and more expensive over time, and there is no diagnostic that says why.

---

## The pattern across all five

Every one of these bugs was invisible because it was the *absence* of a signal, not the presence of one.

The linkedin-analyst was not throwing errors. It was just not writing a done file. The catchup saw a missing file and did what it was designed to do: fire the job. The job was doing exactly what it was supposed to do, just more often than intended. No error. No warning. Eighteen sessions.

The Perl alarm was not failing. It was succeeding at the wrong thing — running until the OS intervened rather than running until the timer fired.

The zombie agents were not crashing. They were running perfectly. They just were not supposed to be running anymore.

The BOOT load was not failing. It was successfully loading every entry in a file that had grown to 103,000 characters without anyone noticing.

Silent automation bugs are the hardest to catch because you have to audit for what is missing, not what is broken. Done files, timeout survival after exec, self-disable path validity, context size — none of these have natural error states. They need explicit contracts and periodic verification.

---

## What the contracts look like now

**Done-file contract:** every runner writes a done file regardless of exit code. The done file records the exit code and timestamp. The catchup reads both.

**Timeout contract:** every Claude session invocation uses `timeout N claude -p ...`. The timeout is the watchdog. The job itself does not own its own termination.

**Self-disable contract:** LaunchAgents that are meant to fire a fixed number of times get an external unloader — a separate launchctl job that checks the firing count and unloads the agent. The agent's own process is not trusted to clean itself up.

**Context contract:** BOOT loads a bounded summary file. Full knowledge files are available but not preloaded. Size cap is explicit in CLAUDE.md.

**autoCompact contract:** enabled by default in settings.json. No session accumulates unbounded context.

Exit code 0 does not mean the work happened. Exit code 1 does not mean nothing happened. Every daemon needs its own done-file contract. Every Claude session invocation needs a real external watchdog.

---

## Then Vera ran an eval

After applying the fixes I ran a structured adversarial eval on the session. Two issues came back.

**The `timeout` fix would have broken the runner entirely.** macOS does not ship with the `timeout` command. `brew install coreutils` provides `gtimeout`, but that was never installed. The fix I applied — `timeout 1800 claude -p ...` — would have hit `bash: timeout: command not found` at 2:00 AM and exited 127. Claude would not have run at all. The fix was correct in intent and wrong in environment. I replaced it with a bash background watchdog that has no external dependencies:

```bash
claude -p "..." >> "$LOG" 2>&1 &
CLAUDE_PID=$!
( sleep 1800; kill -TERM "$CLAUDE_PID" 2>/dev/null; sleep 10; kill -KILL "$CLAUDE_PID" 2>/dev/null ) &
WATCHDOG_PID=$!
wait "$CLAUDE_PID"
EXIT=$?
kill "$WATCHDOG_PID" 2>/dev/null
```

The watchdog runs in a subshell. It does not depend on anything outside the bash standard library.

**The BOOT fix was applied to the wrong CLAUDE.md.** The vault system has two CLAUDE.md files: a global one at `~/.claude/CLAUDE.md` and a workspace-level one inside the vault directory. Per Claude Code's precedence rules, workspace overrides global. I had updated the global file to say "do not re-read vault knowledge files." The workspace file still had the full 180,000-character read instructions. On every vault session, the workspace file won and the context load was unchanged. I updated the workspace file to match.

Both of these are the same class of bug as the five above: no error, no warning, appearing to work while the actual behavior was wrong. The `timeout` call succeeds syntactically. The BOOT loads context successfully from the file that takes precedence. Silent.

The eval step caught them before the next 2:00 AM run.

---

## Related

- [I Cherry-Picked a Viral Cost-Cut Post](cherry-picking-the-cost-post.md) — the optimization work that preceded this audit; four levers adopted, three rejected; the savings from those levers were being partially erased by the bugs in this post
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — an earlier audit of the same LaunchAgent system that found a different class of silent failure: agents registered with launchctl that were never actually running because their scripts lived inside iCloud Drive
- [AI Runners That Remember](ai-runners-that-remember.md) — the memory architecture running on top of these same LaunchAgents; the done-file and timeout contracts in this post are what keep those runners from compounding silently when something goes wrong
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — where the LaunchAgent system started: three agents, clean contracts, no zombie problem; the complexity that created the bugs in this post was added in the second and third automation layers
- [AI Credits Are Infrastructure. Start Treating Them That Way.](ai-credits-are-infrastructure.md) — the next chapter: once the automation bugs were fixed, the question became where the remaining spend was actually going and how to think about it as a PM

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

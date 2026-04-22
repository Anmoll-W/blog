<!-- source_session: 2026-04-21_claude-code-os-upgrade -->

# The Boot Sequence Was in the Docs. So Was Every Skipped Step.

*2026-04-21 · claude-code, automation, vault, ai-tools · Vault as OS*

I analyzed 104 Claude Code sessions from the last eleven days and found the same three failures appearing across every project type, every task size, every day of the week.

Boot sequence skipped. Shallow first pass. Wrong format on the first try.

Not occasionally. As a baseline.

---

## What the Sessions Showed

I had built a CLAUDE.md that laid out everything Claude needed to do at session start: load active-context.md, read past-mistakes.md, check the anmoll-profile, scan automation logs. The instructions were clear. They were tested. They had been working — when I remembered to reference them in my prompt.

The sessions where I forgot — or started fast, or jumped straight to a task — showed the same pattern every time. Claude skipped the boot and went straight to work. The work was technically correct. It was missing context. The same mistakes resurfaced that past-mistakes.md had documented. The same format mismatches that the profile flagged showed up again.

Three patterns, mapped across 104 sessions:

**1. Boot sequence skipped.** Claude jumped into tasks without loading past-mistakes.md or the profile. Not because the rules weren't written — because nothing was forcing them to load.

**2. Shallow first passes.** Tasks were declared done before a proper audit. Missing files, stale cross-references, incomplete link updates. The audit step existed as a CLAUDE.md instruction. It ran when sessions were unhurried. It was the first thing to fall off under pressure.

**3. Wrong format on first try.** An essay when I wanted a meme. A full analysis when I wanted a three-line answer. The format check was documented. It was not being applied before writing started.

The root cause was the same across all three: behavioral rules in CLAUDE.md are text Claude can skip. They're documented, not enforced.

---

## The Product Decision Under the Technical Fix

There is a version of this problem that gets solved by adding more instructions. A longer CLAUDE.md. Better-worded rules. More explicit format examples. More follow-up prompts asking whether the boot ran.

I had tried all of those. They produced marginal improvement. The failure mode persisted because the mechanism was wrong, not the wording.

A rule that runs when remembered is not a rule. It is a suggestion that looks like a rule.

The fix was to move the boot sequence out of documentation and into infrastructure — a Python script that fires via Claude Code's hooks system at every session start and loads the required context as a systemMessage before any task begins.

The difference: one runs when you think to ask for it. The other runs every time.

---

## What Got Built

**SessionStart hook** — `~/.claude/session-loader.py`

A Python script registered as a PreToolUse hook in Claude Code's hooks system. It fires at every session start, reads four sources, and injects them as a systemMessage:

- IN FLIGHT: active projects from active-context.md
- PAST MISTAKES: the Active section of past-mistakes.md, never truncated
- ANMOLL PROFILE: anmoll-profile.md, loaded in full
- AUTOMATION LOGS: today's vault runner logs

Priority-aware truncation: if space is tight, past-mistakes.md survives. It is the section most likely to prevent a repeated error, so it gets protected before anything else is dropped.

The boot sequence that used to run when I remembered to reference it now runs on every session. The instructions did not change. The enforcement mechanism did.

**5 Workflow Rules added to `~/.claude/CLAUDE.md`**

After fixing the boot sequence, I audited the other two failure categories and converted the most-violated instructions into imperative, measurable rules:

- Self-Verification: second audit pass after every multi-file task
- Content Drafts First: list drafts folder before generating any content
- Local-First for Code: localhost confirmed before any PR
- Data Before Doc: live data pulled before writing any analysis
- Format Check: state format (meme/short/essay/thread) before writing

Each rule is written so the check is a binary — either it ran or it did not. Not "consider checking." Run the check, then continue.

**5 global slash commands at `~/.claude/commands/`**

The multi-step workflows that were most likely to miss a step got converted into slash commands:

- `/log-session` — summarize → daily note → decisions → session file → push
- `/linkedin-publish` — drafts check → Vera score → publish → move file → log
- `/check-automations` — silent if green, detailed on failures
- `/cro-pull` — GA4 funnel pull → baseline compare → flag drops
- `/vault-audit` — orphaned files, stale refs, missing indexes

Vera ran a structured review of all five commands. She found six issues: a priority-aware truncation bug in the Python script, orphan-check scope in vault-audit, a grep exclusion pattern, git path quoting, slug derivation, and self-verification specificity. All six were fixed before anything shipped.

---

## What the Second Pass Found

Vera's initial review caught six issues. After those were fixed, a second structured pass ran against the full system — not just the commands, but the hooks, the routing logic, and the verification rule itself. It found three gaps the first pass had missed.

**Deprecated skill names weren't blocked.** Claude had been invoking `superpowers:brainstorm` and `ui-ux-critique` — both deprecated. In one case, this caused Claude to create a CLAUDE.md file instead of running a Vera eval. The fix was a second PreToolUse hook in `~/.claude/settings.json` that fires before any `Skill` tool call. If the invocation matches a deprecated name — `superpowers:brainstorm` (correct: `superpowers:brainstorming`), `ui-ux-critique` (correct: invoke Vera directly), `superpowers:write-plan` (correct: `superpowers:writing-plans`) — the hook outputs a hard warning with the replacement name and blocks the call until the correct name is used.

**Persona routing wasn't loading early enough.** The routing map existed in vault CLAUDE.md. It was not being injected at session start. The fix: `session-loader.py` was updated to output a fifth section, `=== PERSONA ROUTING ===`, containing a compact trigger-word map for all personas, with Vera's signals listed first. Every session now starts with the routing map visible before any task runs.

**Self-verification was too generic to enforce.** "Run a second audit pass" is a rule that can mean anything. It was expanded into four domain-specific checklists — vault tasks, code tasks, content tasks, and a count check. The count check addressed a specific incident: Claude had synced 19 of 24 LinkedIn posts and logged results without flagging the gap. The rule now requires: if the task involved N items, verify the count matches before reporting done.

The meta-finding: the system built to improve verification needed its own verification pass — and that pass found real gaps. The self-referential nature of it is not ironic. It is the point.

---

## The Relatable Part

I had told myself the instructions were good. They were good. Good instructions and enforced instructions are not the same thing.

This is the same failure mode I had documented in the agent system — where Quinn's sign-off existed as a rule and ran zero times in six days, where Vera's ship-readiness eval was written clearly in the protocol and got skipped every time a fast session ended. In each case, the documentation was thorough. The mechanism was missing.

I had not noticed that the same gap existed in Claude Code itself — not in the agents, but in how every session with Claude started. The rules were there. The hooks were not.

---

## What I Learned

**Technical lesson:** The difference between "I asked Claude to load context" and "Claude loads context every session" is the difference between a CLAUDE.md instruction and a SessionStart hook. Instructions remind. Infrastructure enforces. If a step is critical enough to document, it is critical enough to wire into the session start.

**Decision lesson:** When a rule fails more than twice in the same way, the rule is not the problem. The mechanism is. Rewriting the rule will not fix a mechanism gap. The right response is to move the enforcement one layer down — from text to tooling, from suggestion to hook.

**Verification lesson:** Even infrastructure-level fixes benefit from a structured eval pass. The hooks, the routing injection, and the verification rule itself each had gaps that a first review missed. Building the eval habit into the workflow is not overhead — it is what separates a system that holds from one that looks complete until it fails.

---

## Related

- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — the same gap at the agent layer: protocols documented as expectations, not enforced as session gates. This post is the Claude Code equivalent of that session's finding.
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — how past-mistakes.md and anmoll-profile.md were built from 60 sessions of history. The SessionStart hook loads both of these on every session.
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — Vera's structured review caught six issues in the new commands before anything shipped. The same eval posture applied here as to content and architecture decisions.
- [The Vault Was Organized. The Files Were Not.](vault-structural-drift.md) — documented structure enforced by documentation alone drifted for six weeks. The same lesson applied to Claude Code session behavior: if it isn't automated, it isn't running.
- [The Dashboard Was Lists. The Hub Is a Board.](kanban-board-as-project-hub.md) — the same enforcement-by-infrastructure principle applied to task visibility: a Dataview dashboard that required reading replaced by a Kanban board that shows project state at a glance.

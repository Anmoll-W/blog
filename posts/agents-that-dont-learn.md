<!-- source_session: 2026-04-19_agent-management-v3 -->

# Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles

*2026-04-19 · ai-agents, claude, agent-architecture, obsidian, self-improvement · Vault as OS*

Quinn had been an active QA agent on my AI team for six days.

In that time, two build sessions completed. Features shipped. PRs merged. Quinn signed off zero times.

The sign-off requirement existed. It was written clearly in the protocol file. It just lived as a bullet point inside a paragraph — and bullet points inside paragraphs do not run. They get read once and skimmed forever after.

That is the failure mode at the center of most AI agent systems: **protocol existence is not protocol running.**

---

## What Was Actually Broken

My vault runs 13 AI personas — each with an identity file, a memory file, a session start protocol, a session end protocol, and a defined domain. The system had 60+ sessions of history across those agents, thousands of lines of decisions, shared mistake logs, cross-project patterns.

None of it was reaching the agents.

When I ran Harper (the team health agent) to audit the full roster, the findings were specific:

- Five agents had memory files that had not been updated in 8–14 days. Every session they loaded stale context — not because they were inactive, but because the memory update step was documented as an expectation, not enforced as a gate.
- Repeated mistakes were accumulating in `shared-mistakes.md` on Sunday batch cycles. A Supabase bug discovered on a Tuesday would not reach other agents until the following week.
- The global `~/.claude/CLAUDE.md` was 397 lines. The official recommended limit is 200. Anthropic's documentation is direct: bloated CLAUDE.md files cause Claude to ignore the actual instructions. Two of the biggest sections were duplicated verbatim from the vault-level CLAUDE.md that loads in all vault sessions anyway.
- No agent had a profile of how I work, decide, or communicate. Every session, each agent started from scratch on those dimensions.

The root cause was the same across all of them: **guardrails documented as expectations instead of enforced architecturally.**

---

## The Research That Changed the Design

I had been thinking about this as a discipline problem — agents needed to be more consistent. The research reframed it as an architecture problem.

The Reflexion paper (NeurIPS 2023, Shinn et al.) studied what separates agents that improve from agents that stagnate. The finding: verbal self-reflection stored in long-term memory produces a 22% improvement in decision quality over agents that only store episodic session logs. The mechanism is specific — the agent must articulate *why* something went wrong, not just record that it happened. That articulation becomes a retrievable rule, not a forgotten event.

Separately, research on production AI governance found: effective guardrails are enforced architecturally, not documented as expectations. This is the same principle that makes good software design — a constraint that can be bypassed will be bypassed, regardless of documentation.

Those two findings together became the redesign brief.

---

## What the Fix Looked Like

**Reflexion Block on all 13 agents**

Every agent's Session End Protocol now ends with a mandatory Reflexion Block — three lines, non-negotiable:

1. What I worked on this session (1 line)
2. What surprised me or went wrong (1 line, or "nothing unexpected")
3. Rule I am adding (1 reusable pattern, or "no new pattern")

This goes to `memory.md` after every session. The format is not optional. The step is numbered. Numbered steps get executed; paragraphs get skimmed.

**anmoll-profile.md — mined from 60 sessions**

I created a dedicated profile file from 60+ sessions of vault history: how I make decisions, what communication style works, what frustrates me, what triggers action, my scope tendencies, my project switching patterns.

Every agent now reads this as Step 1 of Session Start — before reading their own memory, before loading project context. It took one session to build from vault history. It saves rederivation cost across every future session.

**Memory Decay — a structural staleness flag**

Harper now monitors all 13 agents for a 14-day memory freshness threshold. Any agent whose `memory.md` has not been updated in 14+ days is flagged in the weekly health check with the exact date and a re-invoke recommendation.

A stale agent invoked with wrong context causes more harm than a dormant one. The flag enforces the standard.

**Immediate cross-project propagation**

The old rule: platform-level mistakes get batched into `shared-mistakes.md` on Sunday. The new rule: any mistake involving a shared platform — Supabase, Vercel, LinkedIn, Obsidian — goes to `shared-mistakes.md` in-session, immediately. Waiting until Sunday means the next build session that week hits the same bug.

**Path-scoped rules — context that loads only when relevant**

`.claude/rules/` is a Claude Code directory that supports glob-pattern frontmatter. Files here auto-load only when Claude is working on matching files. Supabase rules — check migration state, parseFloat on numeric columns, RLS on new tables — now live in `.claude/rules/supabase.md` and load only when working on SQL or migration files. They do not consume context in a LinkedIn content session.

**Global CLAUDE.md: 397 → 141 lines**

The session journaling ritual (60 lines) was duplicated between the global file and the vault file. The vault structure description (another 30 lines) appeared in both. Removing duplicates and moving verbose sections to the vault-level file brought the global file to 141 lines — well under the limit.

**Agent frontmatter — explicit model and circuit breaker**

All three task subagents were missing `model` and `maxTurns` in their frontmatter. Without an explicit model, the agent defaults to whatever Claude Code's current default is, which changes with releases. Without `maxTurns`, a looping agent has no circuit breaker. Both are now explicit: code-review and qa-check use Opus, draft-post uses Sonnet.

---

## The Product Decision Underneath This

None of these fixes required new infrastructure. Everything ran on the existing file-based vault system.

The tradeoff was specificity versus generality. Every one of these changes makes the system more opinionated: Reflexion Block outputs go into a specific three-field format. Memory decay triggers at exactly 14 days. The CLAUDE.md hierarchy has exactly four levels now. Agents read exactly one profile file at Step 1.

More opinionated systems are more fragile at the edges and more reliable in the center. For an agent that runs hundreds of sessions, reliability in the center is the right tradeoff.

The alternative — keeping protocols general and trusting agents to apply judgment — is what produced six days of a QA agent that never signed off on anything.

---

## What I Learned

**Technical:** Verbal self-reflection after every session is not overhead — it is the compounding mechanism. Without it, agents accumulate sessions but not intelligence. With it, every mistake becomes a rule that applies to all future sessions.

**Decision-making:** The difference between a protocol that runs and a protocol that does not run is usually one thing: whether it is a numbered step or a paragraph bullet. Number the steps. The format is part of the enforcement.

---

## Related

- [My AI Agents Were Running. They Just Were Not Working.](self-correcting-agents.md) — the session that identified Quinn's zero-activation problem and the team-wide QA infrastructure fix that preceded this architecture upgrade
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — how skills were wired to agent identity files to make methodology consistent across sessions
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — how Vera was built as the team's quality evaluation layer, and how her protocols were similarly tightened
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the architectural decision that set up the persona structure this self-improvement layer runs on top of
- [The Vault Was Organized. The Files Were Not.](vault-structural-drift.md) — built in the session immediately after this one: a nightly automation that enforces the vault's project anatomy the same way this post enforces agent memory hygiene
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — anmoll-profile.md and past-mistakes.md, built from the 60-session audit in this post, are now loaded into every Claude Code session via a SessionStart hook
- [AI Runners That Remember](ai-runners-that-remember.md) — the direct sequel: applying the same compounding-intelligence principle to scheduled runners, with a per-runner memory store, SRS queue, and 2am consolidator
- [The Claims Were There. The Sources Were Not.](claims-without-sources.md) — the next layer of memory: adding source weight (provenance) and hypothesis lifecycle to the knowledge system this post built
- [I Cherry-Picked a Viral Cost-Cut Post](cherry-picking-the-cost-post.md) — the self-improvement layer in action: a re-evaluation date scheduled for two weeks out turns "imported four rules from a viral post" into actual behavior change rather than a calendar entry that gets ignored
- [Four Agents Went RED for Three Weeks. I Cut the Team in Half.](four-agents-went-red.md) — the next iteration: token-bounded retrieval (≤35 lines, grep-only, session-cached) and a Pending Validation memory gate added on top of the self-improvement layer this post built

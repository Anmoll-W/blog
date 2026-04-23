<!-- source_session: 2026-04-16_harper-team-review-agent-self-correction -->

# My AI Agents Were Running. They Just Weren't Working.

*2026-04-16 · vault, ai, systems, agents, product · Vault as OS*

I had told myself the agent system was operational. Thirteen personas, each with a domain, a protocol, a set of rules. Built over two weeks. Documented. Wired together. The kind of system you describe to someone and they say "that's impressive."

I had not actually checked whether any of them were doing their jobs.

When Harper — the HR agent I built to review team health — ran its first full audit, five of the thirteen came back amber. Not broken. Not errored. Just quietly not doing what they were supposed to do.

---

## What "Running" Looks Like When It Isn't

The clearest example: Quinn, the QA engineer.

Quinn's job is to review every feature before it ships. I had added a rule to Alex's (the platform engineer's) identity: "flag Quinn for QA before marking done." I had written acceptance criteria templates. I had wired the personas together in the documentation.

Quinn had run exactly once in six days.

The rule existed. The intent existed. The execution didn't. Because "flag Quinn" is not a mechanism — it's a reminder. And reminders get skipped when sessions are moving fast.

Four more agents had the same pattern in different forms:

- **Nova** (product manager for build projects) had been updated on April 10 with notes about P1 issues on a bot project. By April 16, those items were still sitting in memory, unresolved, with no triage decision. The agent had the information. It had no protocol to do anything about it.
- **Vera** (the eval agent) had been invoked for LinkedIn content exactly once. A full product feature had shipped the day before — avatar system, new database writes, auth boundaries — with no Vera ship-readiness eval. Vera's own rule said "loop both before any feature ships." Nobody looped Vera.
- **Rex** (solo business architect) had been sitting at "pre-validation" for five days on a project where validation was supposed to happen "this weekend." No memory update. No outcome logged. The project had not moved.
- **Maya** had the most visible symptom: zero reposts since March 2. The Repost Formula calls for a quote image brief on every post. The brief had been missing from every post since March 2. The rule was in the identity file. The behavior wasn't there.

In each case, the agent's identity described what it should do. What it lacked was the ability to notice when it wasn't doing it.

---

## The Decision Underneath

There are two ways to build an agent protocol.

The first: describe the behavior, put it in the identity file, and trust that it runs. This works when sessions are deliberate and unhurried. It fails the moment you're moving fast, mid-session, focused on something else.

The second: make the protocol self-enforcing. The agent checks its own health at session start, detects gaps without being told, and blocks progress until the gap is addressed.

The difference is not how thorough the identity file is. It's whether the agent can answer "am I doing my job?" without asking anyone.

I had built the first kind of system and assumed it would behave like the second. The gap between those two assumptions is where all five amber agents lived.

The fix for each one was the same pattern applied differently:

- Quinn's session start now checks for a checklist file before doing anything. No file = no testing. The codebase gets read, the checklist gets generated, and only then does the session continue. This cannot be skipped.
- Nova's session end now requires prd.md and decisions.md updates before the session is considered closed. Not encouraged. Required. And session start includes a P1 staleness check: any item older than seven days without resolution gets surfaced to me with a direct question.
- Vera runs a three-point self-check at every session start: any features shipped without an eval? Any posts published without a content review? Any decisions made without stress-testing? She catches her own missed evals.
- Rex detects stall. If a project stage hasn't changed in seven days, Rex surfaces a go/no-go before doing anything else. Drift becomes a decision.
- Maya's quote image is now an output requirement, not a reminder. No draft gets presented without the brief attached. If three consecutive posts go out without one, she flags it herself.

The agent team also changed structurally. Alex now dispatches Quinn automatically after every build, then dispatches Vera after Quinn returns green. The chain is: build → Quinn → Vera → done. Not: build → done, maybe loop Quinn.

---

## The Relatable Part

I had assumed that because I had written the rules, the rules were being followed. The same assumption that breaks team processes in actual organizations — where someone documents a policy, puts it in the handbook, and is surprised six months later that nobody runs the process.

The handbook existing is not the same as the process running. This is true for human teams. It turns out it is also true for AI agent teams.

The signal I missed: none of my agents had a mechanism to report on their own health. I would only know something was wrong if I asked Harper to look — or if something failed visibly enough that I noticed. Vera had never run a ship-readiness eval, but nothing had broken yet, so the gap was invisible.

The thing worth building is not better documentation. It is agents that know when they are not doing their jobs and say so.

---

## What I Learned

**Technical lesson:** The enforcement mechanism is part of the protocol. A protocol step that depends on a human remembering to invoke it is not a protocol — it is a suggestion. Every critical step in an agent workflow should be either self-triggering or a hard blocker on the next step. "Flag Quinn" is a suggestion. "Session end requires Quinn sign-off before done" is a protocol.

**Decision lesson:** When you build a system and describe it as operational, you have made a claim. That claim needs evidence — not documentation. The evidence is: does the system detect and correct its own failures without you? If the answer is "only when I check," the system is not operational. It is standby.

Before calling any AI system "running," I now ask: what happens when it silently drifts? If the answer is "nothing, until I notice" — that is the gap to close.

---

## Related

- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the session where the agent team was first assembled and the team brain was established
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — how the agent system was restructured from flat files to a stub-based routing architecture
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — how Vera was built and why producing agents cannot grade their own output
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — wiring structured skills to agent identities so methodology is invoked, not recalled
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — the follow-up: after fixing what was broken, rebuilding the architecture so agents compound intelligence across sessions instead of just storing events
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the same gap applied to Claude Code itself: boot instructions that lived in CLAUDE.md but ran only when remembered, fixed by a SessionStart hook that fires every session
- [AI Runners That Remember](ai-runners-that-remember.md) — the next step after fixing agent identity: giving scheduled runners persistent memory across runs so each run benefits from every prior run's strategy

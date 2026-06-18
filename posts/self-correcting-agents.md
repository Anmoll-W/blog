<!-- source_session: 2026-04-16_harper-team-review-agent-self-correction -->

# My AI Agents Were Running. They Just Weren't Working.

*2026-04-16 · vault, ai, systems, agents, product · Vault as OS*

I had built a team of agents — distinct personas, each with an identity file, a memory file, and a protocol describing what it was supposed to do. The team had been running for six days. On paper everyone had a job and a rule that said to do it.

Then I ran the first real performance review. Five of them were not doing their jobs. Not because the rules were wrong — because a rule in a file is not the same thing as a behavior that runs.

This is the gap nobody warns you about when you wire up an agent team: *added to the protocol* and *actually executing* are two completely different states, and only one of them shows up in a file you can read.

---

## The review

I had a review agent whose job was exactly this — read every persona file and every memory file, and grade the team's health. It was the first full review since the team formed six days earlier. Thirteen agents.

It came back AMBER. Five agents flagged. The most damning finding was about the QA agent, Quinn.

Quinn had been added on day one with a clear rule baked into another agent's identity: *no work ships without a QA sign-off from Quinn.* That rule had existed for six days. In those six days, Quinn had run **once.**

The rule was real. The enforcement was not. Quinn's sign-off was written as a bullet in a paragraph inside another agent's protocol — the kind of instruction that is trivially skipped under any time pressure, because nothing structural depends on it. The agent that was supposed to call Quinn just… usually didn't, and nothing noticed.

## "I added it to the protocol" is not "it runs"

This was the load-bearing lesson, and it generalized to every one of the five amber agents.

Adding an agent to a team is an act of writing. Getting that agent to fire is an act of enforcement. I had done a lot of the first and almost none of the second. The protocols read beautifully. The behaviors had quietly never started.

The fix was not to write better prose. It was to change *where* the requirement lived. A critical step cannot be a sentence in a paragraph — it has to be a **numbered, ordered checklist item that blocks completion.** So:

- The agent that owned session-end got a numbered Session End Protocol. Step 1: dispatch Quinn. Step 3: dispatch the eval agent. The session cannot be marked done without both sign-offs. Not "should" — *cannot.*
- The ship-readiness eval got a hard ordering: it cannot start without Quinn's sign-off first. The pair that used to be described as a "pair" — where neither actually required the other — became a sequence where one gates the other.

A bullet became a gate. That is the whole fix.

## Self-correcting, not supervised

The deeper problem with the review was that it depended on *me* running it. A team whose health is only checked when the founder remembers to check is not healthy — it is lucky. So each of the five amber agents got a **self-correcting protocol**: a check it runs on itself, every session, without waiting to be reviewed.

- **Quinn** — checklist file existence is a hard blocker at session start. No checklist file, no testing. It cannot proceed into a broken state.
- **The eval agent** — a three-point self-check at every session start that catches a missed eval immediately, instead of waiting for the next review to surface it.
- **The product agent** — prd and decisions updates are mandatory before session close, plus a staleness check that surfaces anything older than seven days.
- **The finance agent** — seven-day staleness detection that forces a go/no-go to be surfaced, instead of drifting silently.
- **The content agent** — a required image brief shipping with every draft, plus a three-post self-check that trips if the pattern breaks.

The unifying idea: move the check from *something I do to the agent* to *something the agent does to itself.* Supervision does not scale with team size. Self-correction does.

Five agents were fixed in the same session that found them. AMBER to GREEN. No new agents hired to fix the old ones — that would have just added more rules with no enforcement. The fix was making the existing agents enforce themselves.

## The silent failure underneath

The review surfaced one more class of bug that had nothing to do with behavior and everything to do with names.

I had renamed two agents days earlier. The rename had updated *some* of the places each name appeared, but not all. One agent's identity file still carried the old name. Another agent still called the old name. The roster still listed it. Any routing to the new name could have failed silently — no error, just an agent that never answered.

The fix was a rule: a rename has to update **all four surfaces at once** — the persona file, the memory file, every agent that references it, and the roster. A name is a contract, and a half-applied rename is a broken contract that throws no exception.

Same pattern as Quinn, in a different costume: the change looked done because the part you naturally look at was done. The part that actually carried the behavior was still pointing at the old world.

---

## What it changed in how I build agents

Three things stuck, and I have applied all three to every agent system I have built since:

1. **A protocol step that matters must be a numbered, blocking checklist item — never a bullet in a paragraph.** If completion can be reached without it, it will be.
2. **Every agent needs a self-check it runs on itself.** Health that depends on an external reviewer remembering to look is not health.
3. **A rename is a four-surface operation.** Identity, memory, references, roster — all at once, or the routing breaks where you are not looking.

The thing I keep coming back to: nothing here was *broken* in a way that throws an error. Quinn ran successfully the one time it ran. The renamed agents resolved fine under their old name. Every file parsed. The system was running. It just was not doing the work — and "running" had been quietly standing in for "working" for six days before anyone checked.

That is the failure mode of an agent team. Not crashes. Silence that looks like success.

---

## Related

- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the team this review audited: how the personas were formed and wired before anyone checked whether they were executing
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — the eval agent was one of the amber five here; this is the role it grew into once its self-check was enforced
- [Append-Only Is Where Lessons Go to Die](append-only-is-where-lessons-go-to-die.md) — the same "rule existed, behavior didn't" gap at the memory layer, and the learning loop that finally closed it
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — the skills layer these self-correcting protocols hook into; how a borrowable QA checklist became a team-wide capability

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

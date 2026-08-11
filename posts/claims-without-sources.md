<!-- source_session: 2026-05-20_pm-brain-vault-adaptation -->

# The Claims Were There. The Sources Were Not.

*2026-05-20 · vault, knowledge-management, ai-agents, pm, obsidian · Vault as OS*

A Substack post about PM Brain OS landed in my reading queue last week.

The system is a folder of markdown files plus a CLAUDE.md that tells Claude how to use them. Five knowledge areas. Three lifecycle layers. Six commands. The author had built a provenance system: every claim tagged with its source weight. `doc-decision`. `doc-research`. `verbal`. `intuition`.

I read through it and felt something I did not expect: most of this already existed in my vault.

The three-layer architecture (raw ingestion, then working synthesis, then durable knowledge) was already there. The weekly maintenance sweep was already automated, running every Sunday. Session logs, decisions files, pattern tracking: all present.

But the provenance layer was not there. And once I saw the gap, I could not unsee it.

---

## The problem with unweighted claims

My vault had a decisions file. It logged choices made, options considered, reasoning given. 308 lines of durable context.

What it did not log was *how I knew the thing I decided from.*

A founder of a SaaS product says "customers want this feature." A 27-call sales dataset says "this feature was never mentioned as a blocker." Both get logged the same way. Both look equally authoritative when you read them back six weeks later, prepping for a meeting where you need to push back.

That is not a note-taking problem. That is a source-weight problem. The notes are accurate. The weight is invisible.

PM Brain's fix is four tags: `doc-decision` (a written record exists), `doc-research` (data or interviews back this), `verbal` (said in a meeting, nothing written), `intuition` (PM judgment, no external evidence). Simple. No new infrastructure. Just an annotation on every entry.

But there was a flaw in how I initially planned to implement it.

---

## The Vera intervention

Before building anything, I ran the plan through Vera, the eval persona in my agent system whose job is to stress-test decisions before they land permanently.

Vera flagged one issue: the rule I had written said "Claude assigns best-fit provenance if not specified." That sounds reasonable. It is not.

In practice, when Claude infers provenance, it drifts toward `doc-decision`, the tag that sounds most authoritative, the one that matches the act of logging a decision. The result is a decisions file where most entries look like documented choices when many of them were actually verbal statements or PM intuition with no written record behind them.

The fix was to invert the default direction entirely. Instead of "pick the best fit," the rule became: **default to `intuition` unless there is positive evidence in the same session to climb.** A named document gets `doc-decision`. Named data or interviews get `doc-research`. A named meeting gets `verbal`. Without that positive evidence, the tag stays at the floor.

False negatives, where documented decisions look like intuition, are recoverable. You correct them when you notice. False positives, where verbal claims look like documented research, are not recoverable. You act on them.

One rule change. The entire tagging system became conservative by default.

---

## Hypothesis lifecycle: the part PM Brain does better

The second gap was real and had cost me before.

I had patterns.md with strength signals. I had active-context notes. But I had no place where active product bets lived with expiry conditions.

The pattern: a hypothesis gets formed in a session. It gets noted somewhere. Nobody revisits it. Six weeks later it is still influencing decisions, but no one has tried to test it, update it, or kill it.

PM Brain's lifecycle structure is clean: `candidate → proposed → validated → killed`. Each hypothesis carries a `last-evidence` date (what the weekly staleness sweep keys off), a `kill-condition` (the specific thing that would make it false), and a `confidence` level.

The implementation I built adds three fields the base spec lacked: `blocks` (which roadmap decision this hypothesis is gating), `pm-note` (one-line intuition not captured in the evidence), and a two-tier staleness sweep: seven days without new evidence flags the hypothesis as at risk, thirty days auto-archives it to Killed with a reason.

That last design decision matters. A system that only warns you is a system you eventually stop reading warnings from. The auto-kill forces the question: if a hypothesis has gone thirty days without a single piece of evidence, was it ever really a hypothesis or just a wish written down?

---

## Vault-wide from day one

The original plan scoped this to one product. Vera and I agreed: hypothesis tracking for one project is just another orphaned file.

The system now covers four projects: a SaaS product, a Telegram travel bot, a PM learning platform, and a LinkedIn content engine. Each has its own hypotheses.md with seeded bets relevant to that domain. The weekly-review runs a dynamic sweep across all four.

The `/prep` skill, a two-minute ritual before any stakeholder meeting, loads the relevant decisions, active hypotheses, and strategy tensions for the project in question. The most useful step is the conflict-surface check: if an open hypothesis says "X is the bottleneck" but a recent decision reduced investment in X, the prep brief flags it explicitly. You walk into the meeting already knowing where the tension lives.

---

## What PM Brain got right that I had wrong

The system I had was good at capturing decisions. It was not good at capturing *the weight of the evidence behind the decision.*

That distinction matters more in product roles than almost anywhere else. You are constantly in rooms where someone's verbal assertion competes with data you collected six weeks ago. The data is in your notes. The verbal claim is louder. Without source tagging, they look identical when you read them back.

The provenance layer is not overhead. It is the mechanism that makes the notes useful at decision time, not just at archive time.

---

## Related

[Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md): the memory compound problem this builds on: why systems that accumulate sessions without accumulating intelligence eventually plateau

[AI Runners That Remember](ai-runners-that-remember.md): the scheduled automation layer this integrates with: how the weekly-review runner that now sweeps hypotheses was built

[We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md): the Vera intervention pattern: how a decision eval caught a wrong default before it shipped, same mechanism that fixed the provenance tagging rule here

[The Metric Did Not Improve. The Denominator Changed.](the-denominator-changed.md): the same problem in an analytics report instead of a vault. A number gets repeated with confidence because it carries a familiar label, not because anyone checked what it was actually measuring

[The Eval Was Grading My Config, Not My Skill](the-eval-was-grading-my-config.md): the provenance rule applied to eval scores. A pass rate without its file hash and its isolation status is exactly a claim without a source, which is why every published figure in that suite now names both

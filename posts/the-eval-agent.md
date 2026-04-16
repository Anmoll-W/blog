<!-- source_session: 2026-04-13_vera-and-output-measurement -->

# The Eval Agent: Adding a Quality Gate to an AI Workflow

Twelve agents was not the problem. None of them were built to disagree with each other.

I ran my vault with twelve personas — Maya drafts content, Priya specs product decisions, Alex architects the code, and so on. Each agent is good at producing output in their domain. What no agent was built to do was stop and ask whether the output was actually good.

The gap showed up when I measured my LinkedIn posts for the first time. Twelve posts, real impression data. The pattern was consistent: every post skipped the counterargument. Not occasionally. Every one. Maya would write a strong hook, a concrete story, a specific takeaway — and then make a claim in the third paragraph that a skeptical reader would immediately push back on, without addressing it.

I had assumed that a system with twelve specialised agents would catch its own mistakes. It did not. Producing agents and evaluating agents have conflicting incentives, and I had not built any agent whose only incentive was to find problems.

I had a production problem disguised as a content problem.

## Why Producing Agents Don't Catch Their Own Mistakes

There is a structural reason this happens. An agent that produces output and evaluates output is doing two jobs with conflicting incentives. The producing agent is invested in the thing it made. Its goal is to get to "done." Surfacing a flaw means revising, which means more work, which runs against the grain of the goal.

This is not a model limitation. It is an architecture problem. You would not ask a developer to write the code and also write the tests without any separation between those roles. The same logic applies here.

The product decision was to add a thirteenth agent whose only goal is to find problems — and to accept that this adds a step. The alternative was to add an eval checklist to each producing agent. I rejected that for the same reason the self-review argument fails: an agent that produced the output will evaluate it with the same framing that produced it. The eval has to come from a different posture, not a different checklist on the same agent.

I needed an agent whose only job was to look at finished output and say: what is wrong with this.

## What Vera Does

Vera is the thirteenth persona in my vault. Her role is Chief Eval Officer. She does not produce anything. She stress-tests what others produce.

Every eval Vera runs delivers three things, in this order:

1. **Ranked fixes** — what is wrong, ordered by impact, with the reason and the fix
2. **What is solid** — what should not be changed
3. **One blind-spot question** — the thing most likely to be wrong that has not been asked yet

The third item is the one I find most valuable. It is easy to check a list of criteria and catch known failure modes. The blind-spot question forces a different move: what assumption is baked into this output that I have not examined? That is where expensive mistakes live.

Vera runs against three rubric files, each stored at `Knowledge/evals/`:

- `blog-rubric.md` — six criteria for blog and LinkedIn posts (thesis clarity, evidence quality, counterargument coverage, hook strength, teachability, anonymization)
- `decision-rubric.md` — stress-test questions for any product or architecture decision
- `project-rubric.md` — ship-readiness checklist for any project going live

The rule is: read the rubric first, score each criterion, then deliver. No eval without the rubric. This matters because rubrics accumulate patterns over time. After the 12-post analysis, the blog rubric got a new line: counterargument coverage is the most consistently skipped criterion — treat it as mandatory, not optional. The rubric becomes a memory of what has gone wrong before.

## The Session Hook

The other design decision was when Vera fires. It is easy to add an eval persona and never invoke it. The session ends, you are satisfied with what you made, and the eval feels like an extra step.

I added a prompt to the session journal ritual: if this session published a post or made a major decision, ask whether Vera should run a quick eval before closing. Not mandatory — but it has to be asked. The default for most sessions was to skip quality review entirely. The hook makes skipping an explicit choice rather than a passive one.

The first thing Vera did after being onboarded was run an eval on the LinkedIn performance data. She ranked three fixes: add a counterargument before the CTA, replace assertion-only paragraphs with specific numbers, and rewrite CTAs to name a concrete action rather than ask a general question. All three were correct. All three were patterns the producing agent had not caught across twelve posts.

## What the Counterargument Objection Gets Wrong

The obvious objection: can you just review your own work? Read the post again, check it against a list, fix what is weak?

You can, and it is better than nothing. The problem is that self-review tends to catch execution errors (typos, unclear sentences, missing context) and miss structural errors (the claim that sounds strong but is not grounded, the decision that solves the immediate problem but creates a worse one later). Structural errors are invisible to the person who made the structural decision — they are baked into how the thing was framed.

A separate agent brings a different framing. Not a better model, not more intelligence — a different posture. Vera's goal is not to produce something good. It is to find what is wrong. That goal makes different things visible.

## What I Learned

Production and evaluation are separate roles, and mixing them into one agent does not make the system leaner. It makes it blind.

The rubric is the memory. An eval agent without a rubric is just free-form feedback, which decays into opinion. The rubric captures what has gone wrong before, and makes sure the same class of mistake gets checked every time.

The blind-spot question is not optional. Checking known criteria is necessary but not sufficient. The hardest mistakes to catch are the ones that no criterion covers yet — the assumption so embedded in the work that it never got questioned. Forcing that question, even when the answer is "nothing, this looks fine," is the check that matters most.

On decisions: a quality problem in a system with no quality role is not a content problem. It is a staffing decision you have not made. I had twelve agents and no one whose job was to say the thing was not good enough. Adding Vera was not a feature. It was closing an accountability gap that had been open the entire time the system was running. Before adding capability to a workflow, check whether the workflow has any mechanism for catching its own mistakes. If it does not, that is the first thing to build.

---

**[2026-04-13](../README.md#all-posts)** · [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![prompt-engineering](https://img.shields.io/badge/prompt--engineering-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the architecture Vera sits on top of
- [Building an AI Agent Team](building-an-ai-agent-team.md) — the v2 system that preceded this redesign
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the broader automated system this fits inside
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — Vera gains `the-fool` skill in this session: a structured 5-mode critical reasoning framework wired as her primary stress-test tool
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — Vera was one of the amber agents in this review: she had never run a ship-readiness eval despite a feature shipping the day before

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

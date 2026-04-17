<!-- source_session: 2026-04-10_mac-cleanup-and-maya-feedback -->

# Formula Hooks Kill the Metric They Optimize For

*2026-04-10 · ai, systems, content*

I had a rule in my LinkedIn content system that said: "Reframe hooks are the top repost driver." The rule was true. It was based on real data. And it was quietly killing the metric it was trying to protect.

I had not looked at post-level performance in a few weeks. The aggregate engagement numbers were up. What I had not checked was whether the Reframe format was still earning those reposts, or whether something else was carrying the aggregate while the format itself was becoming invisible. When I went back and looked at the last seven posts, six of them had Myth / Reality openers. Repost rate on that format had dropped. I had been watching the aggregate and assuming the formula was still working.

This is not a story about an AI getting something wrong. It is a story about a rule that was correct at the time it was written, then applied at a scale the data did not anticipate, until the rule itself became the problem.

## What Happened

My vault has a marketing agent named Maya. Her job is to draft LinkedIn posts from session notes. She has a skill file that encodes patterns from past performance data. One of those patterns: Reframe hooks (the "Myth / Reality" format) were the top repost driver in historical data.

So she applied them constantly.

The session that surfaced this problem was mundane. I had just done a Mac cleanup, found 166 GB of storage waste, and documented it in the vault. I asked Maya to turn it into a LinkedIn post. She came back with a Myth / Reality hook: "Myth: Your Mac is fine. Reality: You have 166 GB of rot."

It was technically correct as a hook format. It was wrong for the content. The story was personal. It was a confession, not a framework correction. The natural hook was: "I let my system rot for six months. Here is what 166 GB of storage used looks like." That is a scene and an admission. The Reframe format flattened the specificity out of it and replaced it with a pattern the audience has seen 40 times this week.

## Why the Agent Got This Wrong

The skill file had a rule that read something like: "Reframe is the number one repost driver." That statement is true in aggregate, measured at a point in time. But the agent interpreted it as a universal policy: when in doubt, use Reframe.

Two things break when that happens.

First, format-content mismatch. Reframe works when there is a genuinely widely-held myth to flip. It does not work on personal narrative posts. The story of a Mac cleanup has no myth to correct. Forcing the format onto it produces something generic.

Second, the metric self-destructs. Reframe was the top repost driver when it was fresh. When every post is a Reframe, readers pattern-match to it before they finish the first line, and the repost rate falls. The data that justified the rule was collected before the rule was applied at scale. Applying it at scale destroys the data.

This is not a problem with Maya specifically. It is a structural problem in any system where a performance signal gets encoded as a standing rule.

## How I Fixed the Instruction

The decision was not to delete the Reframe rule. The tradeoff: deleting it would have meant throwing away real signal about what earns reposts. Keeping it as-is would have meant the agent continued applying it indiscriminately. The right move was to constrain it and add routing logic — preserving the insight while adding the conditions the original rule had implicitly assumed but never stated.

New rules in Maya's memory:

- Maximum one Myth / Reality hook per week.
- Before defaulting to Reframe, check the last seven posts in the queue.
- Confession and Scene-setter are the default hook types for personal narrative and operational posts.
- Reframe is reserved for PM-framework and opinion posts where a genuinely widely-held belief is being corrected.

The Reframe rule did not disappear. It became a conditional. Apply it when the content type matches and the recency check passes.

## What I Learned

**Performance data is a prior, not a policy.** When you encode "X always wins" into an agent's instructions, you are describing past behavior in a low-competition environment. The rule will be applied in a future environment that the data did not observe. The technical lesson: every performance rule in an agent's skill file needs an expiry condition — a recency check, a frequency cap, or a content-type gate — because the conditions that made the rule true will not hold indefinitely.

**Hook types are perishable.** Format fatigue is real on LinkedIn and every other feed-based platform. A format that earns reposts because it is surprising stops earning reposts once the audience has seen it 20 times. Rotation is not a creative preference. It is how you keep the signal alive. The decision-making principle: if a format has been your default for more than a month, the performance data justifying it is already stale. Check at the post level, not the aggregate.

**Content type should gate format selection.** An agent that ignores content type when choosing hooks will produce technically correct and contextually wrong output every time the content is personal, operational, or confessional. The routing logic needs to be explicit, not implied. Aggregate metrics will not surface this — the format that fails on personal posts may still look fine in the totals if other posts carry the number.

---

**[2026-04-10](../README.md#all-posts)** · [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![content](https://img.shields.io/badge/content-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — what happens when agents execute policy correctly but the policy was wrong
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — the structural fix: an agent whose job is to evaluate output rather than produce it

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

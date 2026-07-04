<!-- source_session: 2026-07-04_fable5-token-audit -->

# The Boot Hook That Refired on Every Compaction

*2026-07-04 · claude-code, token-efficiency, ai-agents, systems · Vault as OS*

I opened a session, sent one message, and watched the context meter read 158,000 of a 200,000 token window before I had asked for anything. No large file attached. No long conversation yet. Just boot.

The instinct is to blame the model, or the task, or the size of the vault. I have written that post before and the answer was never any of those. This time I did not want a guess. I wanted to parse the actual sessions and find the number that explained it.

## Measuring first

I scanned 30 days of usage across 814 session transcripts, 610 distinct sessions. Totals across that window: 81.8 million output tokens, 9.865 billion cache-read input tokens, 423.5 million cache-write tokens, 21.4 million uncached input tokens.

Two numbers stood out immediately. The cache-read figure is about 18 times larger than the other three categories combined, which means almost none of the cost is coming from new work. It is coming from re-reading the same context over and over. And when I broke the 81.8 million output tokens down by type, only 2 percent were prose responses. The other 98 percent were thinking and tool calls.

That second number matters more than it looks like it should, because it rules out the fix most people reach for first.

## What did not work: making responses shorter

There is a popular piece of advice that says the way to cut Claude Code spend is to make the model answer in fewer words. Call it the caveman approach: short sentences, no elaboration, terse output.

Given that only 2 percent of output tokens were prose in the first place, cutting response length to zero would save, at most, 2 percent of output tokens, and output tokens are a small share of the total token bill next to cache-read volume. Run the actual weighting and the realistic saving from response brevity lands around 1.7 percent of cost. That is not nothing, but it is not the lever anyone selling this advice implies it is. Response length was never where the money was going. The money was going into what got reloaded, not what got said.

## The framework: classify by recurrence, not size

Once response length was ruled out, the real question became which injected content was actually expensive. Size alone is a bad proxy, because a small string injected on every single message can cost more over a long session than a much larger string injected only once.

The right way to classify any piece of context a hook or a rule injects is by how often it recurs, not by how large it is:

- Per-compaction — fires every time the session compacts, which happens repeatedly inside a single long-running task
- Per-prompt — fires on every message
- Per-session — fires once when a session starts
- On-demand — fires only when explicitly needed

Optimize in that order. A per-compaction surface that nobody is watching will outspend a per-session surface many times over, purely on frequency, even if the per-session payload is larger on paper.

## Insight: the hook was firing at the wrong frequency

The session boot sequence in this vault runs through a hook that injects context at session start. What the transcripts showed was that the hook was checking whether a session had started, but not checking why. Claude Code's hook payload carries a `source` field that distinguishes a real startup from a compaction event or a resumed session. The hook ignored that field entirely, so every compaction re-triggered the exact same full boot injection meant for a fresh session.

In the prior 30-day window, compaction had already been measured at 243 events. Every one of those 243 events was paying the full boot cost a second, third, or fourth time inside the same task.

The fix was a source-aware hook: full payload only on genuine startup, a 156-character one-line status on compaction or resume. Boot injection on compaction went from 12,501 characters to 156.

## Insight: do not pay twice for the same summary

Separately from the frequency bug, the full boot payload itself — the one that is supposed to load on real startup — carried more than it needed to. It contained complete sections for in-flight work, the last session's summary, the full persona roster, and an all-green status block, all of which were already represented as one-line summaries in the boot header above them. The full expansions were reloading information the summary line had already stated.

Cutting that duplication brought the boot payload from 12,501 characters to 4,349 characters, roughly 3,125 tokens down to 1,087 tokens, a reduction of about 65 percent, on the path that legitimately needs to run at session start.

## Other cuts, same method

A few smaller surfaces followed the same recurrence-first logic. A per-prompt hook injection dropped from 3,537 characters to 323 characters on repeat turns, a cut of about 90 percent, because it recurs on every message and was the highest-frequency surface after the compaction bug. A plugin's session-start skill dump went from 3,253 characters to 1,244 characters, about 62 percent smaller, while every enforcement rule inside it was kept intact. A memory index went from 7,399 characters to 5,770 characters with all 48 entries preserved. Two tool servers defining roughly 240 tool definitions between them were moved from loading on every session to loading only on demand.

I also found subagent definitions pinned to model generations that were no longer current. The fix was to pin explicitly by task shape rather than by an old default: planning and review work on a stronger model, scoped implementation on a mid-tier model, mechanical work on the cheapest tier available.

Fable 5, the newest Anthropic model tier, ran this audit. The task was mostly structured extraction across 814 raw transcripts and disciplined arithmetic on the results, which is the kind of work where a capable model earns its keep by not skipping files, not by writing anything clever.

## The takeaway

Every fix here came from the same discipline: measure the actual transcripts before touching anything, then cut duplication and recurrence, never the substance of a rule. Nothing in the surviving boot payload lost an enforcement rule, a persona definition, or a memory entry. What it lost was the version of itself that got said twice, and the version of itself that fired far more often than the task in front of it required.

If a context surface in your own setup feels expensive, do not start by asking how big it is. Ask how often it fires, and whether anything upstream already said the same thing in fewer words.

## Related

- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — the prior audit that first measured the 243 compaction events referenced here, found in the automation layer rather than the context layer
- [Your Agents Are Only as Good as the Context You Program Them With](context-is-the-program.md) — the boot-token-budget principle this audit extends: that overhaul cut boot context by treating invariants as code, this one cut it by treating recurrence as the real cost driver
- [My AI Agents Got Dumber. It Was Not a Model Downgrade.](why-my-agents-got-dumber.md) — the companion finding on the model-routing side: stale subagent model pins fixed here follow the same right-sizing principle as the persona re-read fix there
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the SessionStart hook this post's source-aware fix was built on top of; this is what happens when a hook that enforces boot behavior does not account for every way a session can begin
- [I Cherry-Picked a Viral Cost-Cut Post](cherry-picking-the-cost-post.md) — the same skepticism applied earlier to borrowed token-saving advice; the response-brevity claim debunked in this post is exactly the kind of lever that framework would have flagged as unverified

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

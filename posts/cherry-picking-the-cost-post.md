<!-- source_session: 2026-04-27_token-efficiency-rollout -->

# I Cherry-Picked a Viral Cost-Cut Post

*2026-04-27 · claude-code, token-efficiency, decision-frameworks, ai-tools*

A post landed in my feed claiming a workflow went from $750 a month to $100 a month on Claude Code with the same coding output. Seven specific levers. Confident framing. Lots of saves.

I was on Max $100 with three days into the billing window and twelve percent of quota used. Sustainable, but if four of those seven levers actually held, my buffer would widen meaningfully.

I adopted four. I rejected three. The framework I used to split them is more useful than any of the levers themselves.

---

## What I kept

**Cache discipline.** Lock tools and model at session start. Mid-session MCP add or remove invalidates the cached prefix. So does swapping model with `/model`. Both force a full re-read of the conversation. The mechanism is documented behavior, not a benchmark — every Claude Code session benefits if you stop fiddling with config mid-flow. Adopted as a hard rule in CLAUDE.md.

**`/effort` per prompt.** A scale from `low` to `max`. Default to `medium`. Reach for `xhigh` only on agentic coding. The lever is per-prompt, not per-session, which is the part most readers miss. Setting `xhigh` once at session start spreads xhigh-grade thinking across every turn that follows, including the trivial ones. Per-prompt scoping is where the savings come from.

**Input format.** PDFs through `pdftotext`, not the Read tool. Read loads PDFs as images, which is the most expensive way to ingest text. Web scraping through `agent-browser` when installed — accessibility-tree based scraping costs roughly an order of magnitude less than screenshot-based browsing. I installed `agent-browser 0.26.0` globally and added it to the rules.

**Subagent model routing.** Plan agent on Opus when the plan involves architectural tradeoffs. Workers on Sonnet for scoped implementation. Explore agents on Haiku for file finds, grep, listing. Crucially, this is set per-Agent-call. It does not switch the parent session's model. Switching the parent would blow the cached prefix and erase the savings.

Four levers. Real mechanisms. Each one verifiable against documented Claude Code behavior, not a claimed benchmark.

---

## What I rejected

**OpenRouter routing to GLM-5.1.** The post claimed GLM-5.1 was approximately Opus-quality at one twelfth the cost. That kind of equivalence claim is a cherry-picked benchmark. Approximately is doing too much work. If I am routing a Plan agent through a third-party gateway to a model with a different reasoning profile, I am not running the same workflow at lower cost. I am running a different workflow with unknown failure modes.

**A code-graph MCP server with a 49× multiplier on context efficiency.** Marketing multipliers from MCP authors are the classic tell. The number is large, the unit is unspecified, and there is no published harness. Ignored.

**An autocompact override.** Disabling autocompact to keep more context alive sounds like savings until you remember the cost of an uncached re-read after compaction is far higher than the cost of letting compaction happen on schedule.

Three levers. Each one a marketing claim or a structural bet, not a mechanism. The cost reduction in the post probably came mostly from the four I adopted, with the other three loaded in to make the number bigger.

---

## The bug the eval agent caught

After installing the four rules, I asked Vera, the vault's eval persona, to run a structured pass.

Vera flagged one rule that nearly broke the rollout: routing the Plan agent to Opus.

The original post was written by someone on Max 20×. On Max 20×, Opus quota is large enough that routing every Plan call through it is fine. On Max $100, Opus has the tightest weekly quota of any tier. If the Plan agent invokes more than three to five times per week, the rule shifts quota toward the most-constrained model. The savings from cache discipline and `/effort` get partially eaten by accelerated Opus burn.

Quota tier is not the same as price tier. Rules calibrated for paying API users can backfire on flat-quota plans. The fix is to narrow Plan-to-Opus routing to architectural plans only and default to Sonnet for everything else.

I would not have caught this without an eval pass. The post does not warn about it because the author was not on this tier.

---

## The framework

When a viral cost-cut post lands, before adopting, ask three questions per lever.

**One — is the mechanism documented or is the number marketing?** Cache discipline, prompt caching TTL, and subagent routing are documented Claude Code behavior. A 49× efficiency multiplier from a third-party MCP is a claim. If the only support for a lever is the post itself, that is a claim. Treat claims as marketing until a harness shows otherwise.

**Two — does the lever apply to your billing tier?** Per-token API pricing, Max 20×, Max $100, and Pro all have different pressure points. A rule that helps an API user with monthly overages may waste quota for a Max $100 user with a tight weekly Opus pool. Same workflow. Different bottleneck.

**Three — what breaks if this rule is wrong?** Cache discipline being wrong costs nothing. Switching to GLM-5.1 being wrong costs unknown reasoning quality on Plan calls — a much harder bug to detect. Reversibility decides how much verification a lever needs before adoption.

Four of seven held under those questions. Three did not.

---

## Where this leaves me

Two assumptions are still not verified. `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` is not in Anthropic's public docs at the time I added it. `/effort` per prompt was sourced from the post, not the official slash command list. Both could be undocumented and silently break in a future Claude Code release. I scheduled a re-evaluation for two weeks out. If either fails, that specific change reverts.

The 20–30 percent quota stretch I am hoping for has no measurement loop yet. Without one, this rollout is faith-based. The next weekly review will record current quota burn rate against the twelve percent / three days baseline. If stretch is under five percent by the re-evaluation date, the rollout did not deliver and gets re-opened.

The framework is the takeaway, not the rules. Cherry-pick. Verify. Schedule a re-evaluation. Treat every viral cost-cut post like it was written for a different billing tier than yours, because it probably was.

---

## Related

- [The Eval Layer Caught Me Violating My Own Rules](the-eval-layer-caught-me.md) — the prior session where the same eval persona caught a different drift; together they show why every system that imports rules needs an eval layer that catches what the rules cannot
- [AI Runners That Remember](ai-runners-that-remember.md) — the runner memory system that this rollout's measurement loop will plug into; the next weekly review surfaces quota burn the same way it surfaces past mistakes
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the same `~/.claude/CLAUDE.md` infrastructure that this post's four rules now live inside; if you have not moved boot from text to hooks yet, start there
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — the self-improvement layer that turns a re-evaluation date into actual behavior change rather than a calendar entry that gets ignored
- [We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md) — the same cherry-pick discipline applied to a full routing architecture: evaluate each claim independently, adopt what fills a real capability gap, skip what's hypothetical
- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — the audit that ran after this optimization rollout; five silent automation bugs that were erasing the gains; what done-file contracts and real timeout watchdogs look like in practice
- [The Boot Hook That Refired on Every Compaction](the-boot-hook-that-refired-on-compaction.md) — the verify-before-adopting rule paying off: the popular response-brevity advice measured out at 1.7 percent of the claimed upside

<!-- source_session: 2026-05-27_pm-credits-optimization -->

# AI Credits Are Infrastructure. Start Treating Them That Way.

*2026-05-27 · ai-tools, product-management, cost-thinking*

The email arrived and I did what most people do: I scanned it, filed it as "billing noise," and went back to work. My AI tooling provider was splitting programmatic usage into a separate credit pool starting June 15. A new line item. A new meter. I had a vague sense that this would matter eventually, so I decided to look at what I was actually spending.

What I found was embarrassing — not because the number was large, but because I had no idea where it was going. I was using AI tooling the way I once used free tiers: carelessly, because it felt like it did not cost anything yet.

The audit took less than an hour. I broke down usage by model tier, by token type, and by session. The shape of the spending was not random. It was three things, stacked.

The first was model routing — or the absence of it. I had been defaulting to the highest-capability model for everything: fetching a file, summarizing a list, looking up a value, generating a paragraph. The most capable model is also the most expensive. The cheaper tiers cost a fraction of it and are genuinely sufficient for the majority of the work. I was using a precision instrument to hammer nails.

The second was session length. Long sessions accumulate context. Every message you send forces the model to re-read that context. In a short session this is negligible. In a session that runs for many hours — the kind of all-day debugging or planning marathon where you just keep going — the re-read cost compounds. It does not feel expensive in the moment. It is invisible. But when you look at the breakdown by token type, context re-reads account for far more than the actual output you cared about.

The third was the automated pipelines. I had built several runners that execute on a schedule: morning briefings, weekly digests, analysis jobs. They worked well. They also ran with full, uncompressed context every single time — context that was accumulated for interactive use, where I was asking follow-up questions and needed the thread. The runners did not need the thread. They needed the data. But nobody had told them that.

Three levers fixed most of it. Running a compaction command periodically in long interactive sessions — it collapses the context to a summary without losing the meaningful thread. Routing by task type — exploratory and creative work stays on the capable model, mechanical lookups and retrieval go to a cheaper tier. Trimming the automated pipelines down to only the context they actually need, not the context they inherited from being built interactively.

None of this is novel. It is the same thinking I would apply to any infrastructure cost. When a cloud spend review shows that expensive compute instances are handling workloads that cheaper instances handle fine, you right-size. When a database is doing full table scans on queries that a simple index would resolve, you add the index. The framework is identical. The only thing that changed is that AI usage now has a bill with enough granularity to apply the framework.

That is the point the Anthropic billing change actually makes. Not that AI tools are suddenly expensive — they were always costing something, the cost just was not visible. The new credit pool is a measurement tool. It surfaces what was previously diffuse noise.

PMs are generally decent at infrastructure cost thinking when the infrastructure is obvious. Cloud spend, API rate limits, database egress — we treat those as engineering constraints worth understanding. The same skills transfer directly. Read the bill. Break it into components. Identify the top three drivers. Apply the smallest change that moves the needle. Verify.

What makes AI spend feel different — what made me file that email as noise — is that the usage is personal and fuzzy. I did not have a dashboard showing my session counts. I was not monitoring token types. It felt like asking a craftsperson to track how many times they picked up a hammer. But the analogy is wrong. AI tooling is infrastructure that scales with behavior. Behavior has patterns. Patterns have costs.

The invisible is about to become visible for a lot of people on June 15. The skills to read it are not new. The only new thing is actually looking.

## Related

- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — the audit that surfaced the automation bugs quietly compounding cost in the background; where the invisible spend first became visible
- [We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md) — the model routing decision that came before the cost breakdown; why the cheapest sufficient model is a design choice, not a compromise
- [One Sheet I Can Trust](one-sheet-i-can-trust.md): a pipeline where every paid surface has a hard cap by design, and the free-credit math decided the architecture

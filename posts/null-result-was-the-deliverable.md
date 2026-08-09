<!-- source_session: 2026-07-28_product-blog-batch -->

# I ran the stats hoping to prove it worked. It did not, and that was the deliverable.

*2026-07-28 · ai, statistics, experimentation, product-management*

For months, a SaaS product I work on went through a series of funnel redesigns. Each version shipped to all of the traffic, all at once. No control group, no held-out segment, no A/B split. We built a change, we deployed it, we moved to the next one, each time telling ourselves a growth story about why the new version was working. Eventually the roadmap question stopped being avoidable: were any of these redesigns actually worth the engineering time they had cost, and should we keep iterating on this same funnel or spend that time somewhere else. Answering it meant going back over eight distinct historical windows, stretching from before the first redesign through the version that eventually settled into place.

My working assumption going in was simple, maybe naive: if I pulled the raw numbers and ran a proper statistical test instead of eyeballing percentage changes, I would get a clean yes or no to bring back to that roadmap conversation. Conversion rate either went up in a way that could not be explained by noise, or it did not. I expected the exercise to be quick and produce a tidy verdict I could put in a report.

That is not what happened.

**Picking a method I could actually defend**

The first problem was that the story we already had, the one built from prior version-by-version write-ups, traced back to a funnel-scoped script that no longer existed on disk and whose methodology nobody could fully reconstruct. Rather than try to re-derive something undocumented, I went a different route: pull raw sessions and key events for Organic Search and Direct traffic across all eight windows, using a metric this dashboard's own reporting tool could reproduce end to end. Then run two-proportion z-tests on the deltas between windows, at the standard 0.05 significance threshold.

Six tests came out of that. Comparing the earliest pre-redesign window against the final settled version, both channels came back not significant, and for Direct traffic the point estimate actually pointed toward a decline, not an improvement. That was already a quieter result than I expected, but it was the next comparison that mattered more.

When I compared the window immediately before the redesign that ultimately stuck against the settled version, Direct traffic produced the only nominally significant result in the whole set, a p-value that cleared the 0.05 threshold, but not by a wide margin. Briefly, I had my headline: the redesign worked, at least for one channel.

**The result that would not survive a rerun**

Then I reran the same comparison, but moved the baseline forward by one step in time, to the very next window in the sequence, the one immediately before the version that replaced it. Same settled version as the comparison point, same channel, same test. The significant result vanished. Not "weaker." Gone.

That is the moment the actual finding revealed itself. It was not that one baseline was right and the other was wrong. It was that a real effect should not flip depending on which adjacent week you happen to pick as your starting point. A test result that only survives under one specific framing, and fails under an equally reasonable neighboring framing, is not evidence of an effect. It is evidence of instability, and at the sample sizes involved here, single-digit to low double-digit key events in some windows, that instability looks exactly like what pure sampling noise produces. One out of six tests crossing the significance line, and failing to replicate against the very next window over, is close to what you would expect from chance alone.

So the honest bottom line I brought back to the roadmap conversation was not "the redesigns worked" or "the redesigns failed." It was that on the one metric I could independently reproduce from raw data, there was no statistically defensible evidence the changes had moved conversion in either direction, for either channel. That does not mean the changes did nothing. It means the data available could not distinguish "it worked" from "it did not" at the volumes involved, which is its own answer to a roadmap question, just not the one anybody wanted to hear.

Separately, while pulling that same eight-window dataset, I noticed something else: two metrics that had tracked each other closely for months, transactions and keyEvents inside our analytics, started diverging in the most recent of those windows, consistent with a suspected miscount tied to an upsell event that had been over-firing as a conversion. That divergence had been quietly inflating the conversion story in that final stretch of the comparison, which is part of why the pre-existing narrative had looked more convincing than the underlying numbers actually supported.

**Why there was no rescue technique available**

Before accepting "we cannot know," I looked at whether more sophisticated methods could recover a defensible answer retroactively. Interrupted time series and CausalImpact are the standard tools for exactly this situation, a change shipped without a formal experiment. Both work by modeling a counterfactual, what would have happened without the change, using a pre-period. But CausalImpact specifically requires an auxiliary control series, some metric or segment the intervention did not touch, to separate the effect from everything else moving in the background. Because every redesign here went to all traffic with nothing held back, there was no unaffected series left to use as that control. The gap was not a missing analytical technique. It was missing data. No control group had ever existed to analyze.

The fix that came out of this was not a better statistical test. It was a product decision: the next funnel or website change ships with a held-out segment, even a simple split, so that if someone asks "did it work" six months later, there is actually a control to compare against, and roadmap time never again gets spent on a redesign nobody can prove helped.

The lesson is that a null result reached honestly is more valuable than a positive result reached by only checking one baseline, especially when that result is what decides whether a team keeps investing in the same idea. If a finding depends on which adjacent time window you happened to choose, it is not describing the world, it is describing your choice of window, and the discipline that matters most is checking whether your conclusion survives being asked the same question from a slightly different angle before you let anyone act on it.

## Related

- [We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md): the same discipline in a different domain. An assumption that looked settled until one adversarial check was actually run against it
- [The Metric Did Not Improve. The Denominator Changed.](the-denominator-changed.md): a companion case from the same reporting stack. A number that survived because nobody checked what it was actually measuring
- [Analytics Said Zero. The Order System Said Otherwise.](analytics-said-zero.md): the same "check the number against a second source before you act on it" discipline, applied to a very different kind of claim
- [The Guard I Built, Measured, and Deleted](the-guard-i-measured-and-deleted.md): the same measure-before-you-trust move applied to a safety check I had built and believed in, where the measurement said delete it

---

*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

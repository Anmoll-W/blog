<!-- source_session: 2026-06-10_vault-os-2.0-build -->

# Append-Only Is Where Lessons Go to Die

*2026-06-10 · vault, ai, systems · Vault as OS*

My second brain had written down 933 lines of lessons for its AI agents, and the agents were still making the same mistakes. The problem was not capture. The problem was that nothing ever read, ranked, or retired what was captured. This post documents the overnight learning loop I built to fix that — and the three places a safety system saved me from myself.

## Context

My vault runs a team of six specialist AI agents — content, engineering, product, quality, finance, and systems. Each one had a memory file. Every time an agent made a mistake, a lesson was appended. After a few months: 933 lines of memory, 396 archived session logs, 27 mistakes marked high-severity — and the same corrections being given over and over.

Research on agent memory explains why. Append-only lesson logs fail structurally: information buried mid-file suffers heavy recall dropout, and a log with no eviction signal cannot tell a load-bearing lesson from a stale one. Writing things down is not learning. Curation is learning.

## The design: lessons with a score, and a night shift

The rebuild has three parts.

**Lessons became data, not prose.** Every lesson is now one atomic bullet with a counter: how many times it helped, how many times it misled, and when it last fired. Each agent is capped at 50 bullets. When the cap is hit, the lowest-scoring lesson is evicted — where the score punishes misleading twice as hard as helping rewards. The migration distilled those 933 lines into 215 scored lessons.

**Capture became automatic and free.** A small hook watches my prompts for correction language — "no," "wrong," "I told you" — and files the exchange into an inbox folder, raw. A second hook writes a five-line stub at the end of every session. Neither costs a single model token. Crude capture is fine, because the thinking happens later.

**The thinking happens at 2:00 AM.** A nightly job on an always-on server pulls a mirror of the vault, reads the inbox, and proposes lesson deltas — add, strengthen, or evict. A separate deterministic script applies them, enforces the caps, and updates the counters. The model proposes; plain code disposes. The vault syncs through a private git mirror every 30 minutes, so the whole loop runs with the laptop closed.

The next morning, every session boots with its agent's current lessons loaded — and the router that assigns work to agents now scores the request against weighted keyword tiers instead of matching flat keywords, because the old router had misrouted the very session that designed this system.

## What surprised me

**The security gate paid for itself before the first push.** I added a secret scanner that blocks any sync containing credential patterns. On its very first scan of the vault — before anything left my machine — it found a real OAuth secret pasted into a session log from two months ago, sitting in cloud-synced notes the entire time. Twenty-eight findings, twenty-seven false positives after review, one real leak scrubbed at the source. If you sync notes anywhere, scan them. You have pasted a secret into a note. So have I.

**My first simulation of that gate was wrong.** I planted a leaked key to test the blocker and the commit sailed through. The planted key turned out to be the documented example key that scanners deliberately ignore, and my exit-code check was measuring the wrong process. A simulation that cannot fail is not a simulation. The corrected test — a realistic random key — was blocked, alerted, and aborted exactly as designed.

**The AI was not allowed to arm its own pipeline, and that was correct.** Three times during the build, the assistant's own safety layer hard-blocked it: pushing the full vault to the remote, deploying the autonomous job, and sending vault content to the model provider for a test. Each required my keystroke instead. My first reaction was friction. My second reaction was that this is exactly the property you want in a system that ships your private notes somewhere every night — the autonomous paths run on timers and schedules that I armed, and the AI cannot arm them itself.

**The first real run proposed nothing.** When the reflector ran live for the first time, it read the day's signals — which were only file-modification stubs — and declined to write any lessons, reporting there was no concrete behavioral evidence to justify a change. A learning system that refuses to learn from noise was the single best result of the day. The run cost five cents.

## What I Learned

1. A memory system needs an eviction policy more than it needs more memory. Counters and caps did more for my agents than every previous attempt at writing better lessons.
2. Capture and curation are different jobs on different schedules. Capture must be instant, deterministic, and free. Curation can be slow, model-driven, and nightly. Mixing them is how you get append-only graveyards.
3. Test your security gates with inputs that can actually fail, and treat a hard block from a safety system as information, not friction. Both saved me in the same evening.

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — where these scheduled agents were first built; this post is what happened once they could write but never learn
- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — the cost-discipline side of the same system: what preloading everything at boot was costing, and why retrieval had to become just-in-time
- [Every Status Was Green. Three of Them Were Lying.](every-status-was-green.md) — a night inside this learning loop where the model's response failed to parse, the run deleted its own input, and the health log still said ok

---

*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work.*

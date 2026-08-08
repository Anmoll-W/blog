<!-- source_session: 2026-06-02_hermes-wave4-kickoff -->

# Hermes, Wave 4: Finance Intel From a Server That Cannot See My Finances

*2026-06-02 · ai, systems, automation, finance · Hermes as a PA*

Wave 1 gave my vault a body — a small rented Linux server that runs all day, sends me a morning digest, and saves reminders I text it. Wave 2 moved my scattered automation onto that hardened server and taught me, five times over, that a green test suite will confidently call broken code fine. Wave 3 taught the body to draft from my notes without ever being able to post. Wave 4 is where the body learns about money — and immediately runs into a wall I built on purpose.

The new capability, stated plainly: each morning the assistant tells me where I stand against a single long-term goal — a target corpus, a date, a monthly contribution I have promised myself never to cut. It can alert me when a coin on a watchlist moves sharply. I can text it "invested fifty thousand in a Nifty index fund" and it logs that to the vault without me opening the laptop. And on a deeper weekly pass it works out how far I am from the goal at my current pace.

That is the feature. The reason it is worth a post and not a changelog line is the wall.

---

## The Wall I Built Against Myself

From the very first wave, the rule has been that finance data never reaches the server. My vault holds finance notes, health notes, and personal notes, and the sync to the server is deny-by-default: nothing crosses unless I have explicitly named it as safe, and every finance path is named as forbidden. The server holds project notes and daily logs. It has never held a single rupee figure, a holding, or an account.

So Wave 4 sets up a contradiction. I want finance intelligence delivered by the always-on server. The always-on server is the one machine that, by deliberate design, cannot see my finances. How do you get a daily corpus figure out of a body that is blindfolded to the very thing it is reporting on?

The lazy answer is to widen the blindfold — let a little finance data cross, just the bits the feature needs. I have done exactly that kind of thing by accident before. In Wave 3, adding two new sources meant widening the synced slice, and the rule I first wrote was too broad: it would have copied private session notes to the server. The adversarial review caught it before it shipped. That memory is why widening the slice for finance was never on the table here.

---

## The Resolution: Compute Where the Data Lives, Ship Only the Verdict

The answer is to keep the data exactly where it already is and move only the conclusion.

The math runs on the Mac. The Mac is the machine that already sees my finances — it always has, and nothing about that changes. A small program there reads the finance notes, works out where I stand against the goal, and produces one sanitized summary. Then only that summary crosses to the server, through a path that was already allowed. The server reads the summary and delivers it to my phone. It never sees the inputs. It only ever sees the verdict.

It helps to be exact about what "sanitized" means here, because the whole privacy claim rests on the line being drawn in the right place. What never leaves the Mac: the absolute portfolio value, every per-asset number, account and platform names, holdings, position counts, profit and loss. What may cross to the server: whether I am ahead of or behind my glide path and by roughly how much, the percentage of the way to the goal, whether this month's contribution has gone in, an estimate of how long until the goal at the current pace, and a timestamp.

There is one number that does cross, and I want to be honest about it rather than hide it. The single headline corpus figure — one number, in crores, nothing broken down — is delivered each morning, and that figure does transit the server. That was a deliberate decision, mine, logged as a decision: a single aggregate total on a server I personally control is an exposure I accepted, because it is the one number I actually want to see every morning. The balance sheet behind it — what makes up that total, where it sits, what it is invested in — never crosses. The verdict travels. The ledger stays home.

---

## Why This Does Not Corrupt Anything

A pattern like this has a trap waiting in it, and it is the same trap from Wave 1: two machines writing the same file.

The sync tool I use never merges. If two devices change the same file before they sync, it keeps both copies and leaves me a conflict to untangle by hand. The summary file the Mac writes is, in principle, a file the server might be tempted to touch — and if it did, the next sync would produce exactly the kind of conflict that discipline exists to prevent.

The defence is the rule the whole system runs on: every file has exactly one writer. The Mac writes the summary. The server only ever reads it. There is no path in the code where the server writes back to that file, which means there is no race to lose.

---

## Honest, Not Fake-Fresh

The harder problem with a Mac-computed number is that the Mac sleeps. If I am reporting a daily figure but the machine that computes it was closed all night, what number does the morning message show?

The wrong answer is to show last night's number as if it were current. A figure that lies about its own freshness is worse than no figure, because I would make decisions on it believing it was live. So the summary carries a timestamp of when it was actually computed, and two guards sit on top of that timestamp.

On the Mac side, the figure is recomputed whenever the machine wakes and periodically while it is awake, so the morning message gets the freshest value the machine could honestly produce. On the server side, a freshness guard refuses to deliver a stale figure as if it were new — if the most recent computation is more than a week old, it does not invent confidence it does not have; it tells me the figure is stale and that the Mac producer needs to run. Every delivered number carries an "as of" stamp. A close laptop degrades the feature to an honest old number, never a confident wrong one.

There is a small grace in the timing that makes this less fragile than it sounds. The morning message lands before the Indian market opens and while the American market is closed, so "last evening's figure, plus a live read on the part that trades around the clock" is already close to real time at the moment I read it.

---

## The Line That Matters Most: It Never Touches the Money

Everything above is about privacy. The other invariant is about action, and it is the one I care about most.

Hermes cannot trade. It cannot move money. It cannot change my monthly contribution. It cannot make a decision about when or how I draw the corpus down. The entire wave is observe, alert, and phrase — nothing else. The math is done in plain code; the model's only job is to put that math into a sentence I will actually read. The model never decides anything financial, because it is never handed a financial lever to pull.

This is not a promise I am trusting myself to keep. It is a guard — a test whose whole job is to assert that nothing in the finance path can execute a financial action. If some future, lazier version of me ever wires in a "just rebalance it for me" call, that test goes red. The same way Wave 3's human-approve gate is enforced in code rather than left to my good intentions, Wave 4's no-money-movement rule is enforced in code. The discipline does not depend on me remembering it at the moment I am most tempted to cut a corner.

---

## Three Lenses Before It Ships

The build was reviewed through three different lenses before I was willing to call it done, and each one was looking for a different kind of failure.

The first lens was build correctness — does the code do what it says, does it handle the boundaries, does it fall over gracefully. The second was the adversarial lens that has run on every wave: a reviewer whose entire job is to be hostile, to find the leak and the silent failure I did not imagine. The third was new for this wave — a finance-domain lens, asking whether the numbers themselves were honest. Was the way I projected the path to the goal sound? Was the floor I measured against the right floor? That review changed real things: it corrected the baseline I measure progress against and pinned down the conservative assumption underneath the projection.

The adversarial pass ran dozens of probes and surfaced one serious finding plus a handful of smaller ones, all fixed before any of this went live. As with every wave, nothing got changed on the strength of a review alone — each finding was reproduced against the real artifact first, because reviewers are sometimes wrong and breaking working code to chase a phantom is its own failure.

---

## Where It Stands

I will be exact about status, because vague claims are how this kind of post turns into marketing.

Wave 4 is built, reviewed through all three lenses, tested, and — as of today — deployed. The branch is merged, the jobs are on the server, the crypto watcher is on its schedule, the investment-capture hook is loaded, and the Mac producer that computes the corpus figure is running and writing the pulse. The daily corpus line is wired into the morning briefing. It just went live. What it does not have yet is a soak: no stretch of mornings where this figure has arrived on my phone day after day, because it has only just started running. When I write "each morning the assistant tells me where I stand," I now mean that literally — but I have not yet watched it do so for long enough to vouch for it.

What I am deliberately not claiming: any outcome. No time saved, no better decisions, no closer-to-the-goal-because-of-this. Those would be inventions today. They belong in a later note, if they turn out to be true, after the thing has run long enough to have something honest to say.

So the claim for now is only this. The math runs where the data already lives. One sanitized verdict — and a single headline number I chose to expose to myself — crosses to a server I control, which delivers it and nothing else. The body cannot see the balance sheet, cannot move a rupee, and cannot change the one promise the whole exercise is built around. It can only tell me, each morning, whether I am still on track — and let me decide what to do about it.

That was always the point. Not an automated money manager. A body for a brain I already trust, given exactly enough sight to keep me honest about a long-term goal, and no power at all to act on it.

---

## Related

- [Hermes, Wave 3: A Machine That Drafts From My Notes But Cannot Post](hermes-wave-3-the-machine-that-drafts.md) — the previous wave, where the same "enforced in code, not in good intentions" gate kept an autonomous drafter from ever posting; Wave 4 applies the identical posture to money
- [Hermes, Wave 1: Giving My Vault an Always-On Body](hermes-the-foundation.md) — the deny-by-default privacy boundary and single-writer discipline this wave both honours and depends on
- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — the wave that established why green tests are not enough, and why every Wave 4 finding was reproduced before being trusted

---

*This post was distilled from a working session building and reviewing the Hermes Wave 4 finance and work-intel layer. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES
- "Wave 4 = daily corpus figure in morning briefing, crypto market alerts, Telegram investment-capture to vault log, weekly FIRE deep-dive; all alert/format-only, never trades/moves money/changes SIP" — wave4-finance-intel-design.md §Conclusion, §1, §9; index.md §1 "What Hermes will NEVER do", §2
- "finance paths fail-closed out of the slice; server has never held finance data" — index.md §3.1 (.stignore deny-by-default), wave4 design §2 "finance-data paradox"
- "resolution: Mac computes, sanitized freshness-stamped pulse crosses to VPS via already-allowed path, VPS reads+delivers only" — wave4 design §2, §3 (mermaid bridge), Appendix A
- "sanitization rule — NEVER sync: absolute corpus, per-asset, account/platform names, holdings, position counts, P&L; MAY sync: glide status, progress %, SIP status, months-to-FIRE est, milestone label, computed/as-of stamps" — wave4 design Appendix A "Sanitization rule"
- "one exception: single headline corpus_total_cr (crores, 3dp) DOES cross, deliberate logged decision, Anmoll opted in; no per-asset/holdings/accounts" — wave4 design Appendix B "Posture change", §3 "Accepted exposure (logged in decisions.md)"
- "single-writer = Mac on pulse file; VPS reads only; no merge / no conflict" — wave4 design §3 "Why this is conflict-free", index.md §5 single-writer
- "freshness: Mac recomputes on wake + periodically; VPS freshness guard refuses stale >7d, flags instead; every figure as-of stamped; never fake-fresh" — wave4 design Appendix B "Freshness", Appendix A "Freshness guard (VPS)"
- "market-timing grace: 8:30 AM IST Indian market pre-open + US closed so last-evening+live-crypto is near real-time" — wave4 design Appendix B
- "no-financial-action enforced by executable guard test (test_no_financial_action.sh, 12/12)" — wave4 design §5 file map (jobs/test_no_financial_action.sh), Status table MW4-4 "guard 12/12"
- "math in plain Python, LLM only phrases" — wave4 design §2, §Conclusion "Python does the math, the LLM only phrases"
- "three review lenses: build correctness (Alex), adversarial (Vera), finance-domain (Rex); Rex corrected glide-baseline + 3.5% floor; Vera 30 probes found 1 HIGH + MEDs all fixed" — wave4 design Status "BUILT + gate-reviewed", §6 MW4-4b
- "validate before fix — reproduce each finding against real artifact" — Wave 1 + Wave 3 posts (established discipline); wave4 design §7 launch checklist step 5
- "Wave 3 slice-widen-too-broad would have leaked private session notes, caught in review" — hermes-wave-3 post + wave3 design; reused here as precedent
- STATUS: built + gate-reviewed + tested + DEPLOYED 2026-06-02 (PR #2 merged to main, merge 041817c; crypto-watch fd73a59; jobs on /opt/hermes; Mac LaunchAgent com.anmoll.hermes-finance-pulse loaded + pulse file live; corpus line in morning briefing). NO soak yet, NO outcome/time-saved/engagement claimed.
- ANONYMIZED: no LinkWhisper named; corpus/goal stated as concept (target corpus, date, monthly contribution) — exact rupee figures (₹69.7L, ₹3L SIP, ₹5.37Cr floor) deliberately OMITTED per house rules
-->

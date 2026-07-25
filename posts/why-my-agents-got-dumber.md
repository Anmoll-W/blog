<!-- source_session: 2026-06-18_agents-got-dumber-config-audit -->

# My AI Agents Got Dumber. It Was Not a Model Downgrade.

*2026-06-18 · claude-code, ai-agents, token-efficiency, systems · Vault as OS*

The agents felt slower to reason. Subagents I dispatched came back shallower than they used to. My first instinct was the obvious one: something upstream got quietly downgraded. A smaller model behind the same name.

I ran an audit before believing that. Nothing upstream had changed. Every cause was sitting in my own config — optimizations I had added earlier to save tokens, each one quietly trading away intelligence I had stopped noticing I'd traded.

This is the post I'd want to read before blaming the model.

---

## Culprit 1: I had routed my agents to be dumb on purpose

Earlier I had adopted a subagent routing rule: send the cheap work to cheap models. Implementation workers went to a mid-tier model. Search and file-finding went to the smallest one. The main session stayed on the top model.

On paper that is sensible. In practice I had written a blanket rule and then forgotten it existed. *Every* agent I dispatched was running a weaker model than the session that dispatched it — including agents doing real reasoning that happened to be labeled "worker." The agents were not getting dumber. They had been configured dumb and I had stopped reading the rule.

## Culprit 2: My default effort was set to think less

The second lever was reasoning effort. I had set the per-prompt default to a middle tier — explicitly chosen because it was a large token saving over the high tier. The note in my config even said so.

Middle effort thinks less. That is the entire point of it. For mechanical work that is correct and free. For anything with judgment in it, I had quietly defaulted myself into shallower reasoning and called it efficiency.

## Culprit 3: The expensive one — re-reading the same file every single turn

This is the one that actually surprised me.

My setup auto-routes each message to a "persona" — a role file plus a lessons file that shape how the assistant evaluates the work. The routing hook injected an instruction on every routed message: *read the role file and the lessons file before you respond.*

Those two files are roughly 3,700 tokens. The content does not change within a session. And I was re-reading all 3,700 tokens on every single routed turn. On a long session that is the same unchanging text reloaded dozens of times — pure waste, and worse, it crowded the context window that the actual work needed.

---

## The fix is not "turn everything back up"

My first patch was the lazy one: route every agent to the top model, default effort to high. It worked, and it was also wrong — just the opposite mistake. Paying the most expensive model to run `grep` is as dumb as running hard reasoning on the cheapest one.

The real fix is **right-sizing**, and it has to be a judgment per task, not a blanket:

- **Smallest model, low effort** — provably mechanical work where being wrong is obvious and cheap to redo. List files, count, rename, reformat.
- **Mid model, medium effort** — scoped execution against a clear spec. The *what* is decided; the agent just does it.
- **Top model, high effort** — reasoning, ambiguity, architecture, anything customer-facing, anything where being wrong is expensive.

The tiebreaker that matters: **route up when unsure.** A weak agent on a hard task wastes more time and money than the model you saved on. Route down only when the task is mechanical *and* verifiable *and* cheap to redo.

That single rule is cheaper than blanket-top *and* smarter than blanket-cheap. Both blanket modes were the bug.

## Fixing the re-read without making it dumb

The persona re-read had an obvious fix — load each role once per session, then reuse it. I built exactly that: a tiny state file remembers which roles have loaded, and the hook only re-injects the full read when a *new* role is needed or you switch to one not yet loaded. Switching roles still loads each fresh. Staying on one never re-reads.

Then I caught the subtlety that would have quietly broken it.

"Load once, then reuse" assumes the file is still in the context window. But a long session **compacts** — it summarizes and drops older content to make room. If a role's lessons got evicted by compaction, my hook would still say "already loaded, reuse it" and I'd be operating on a role I could no longer actually see. Cheaper, and silently dumber. The exact failure I was trying to kill.

The right fix is mechanical, not hopeful. Context only gets evicted by three events — compaction, clearing the session, and session reset — and each one fires a hook. So I clear the "loaded" flag on all three. If the guidance left the window, the flag is gone, and the next turn reloads the full file. Steady state stays cheap; the full reload happens only when the content was actually dropped.

The principle I wrote on it: *nothing should get dumb, and nothing should overpay.* I rejected a tempting alternative — inject a short summary every turn — because it cost tokens forever to defend against an event that already has a hook for it.

---

## The cost I have not paid down yet

The audit found one more thing, and honesty matters here: I have measured it but not cut it. My setup loads **246 skills across 12 plugins** into context — **129 of them from a single plugin** I rarely use, plus nine tool servers whose instructions load whether or not I touch them. That is the largest fixed overhead in the whole system, and it is sitting there on every session start.

I am calling it out as identified-and-pending rather than claiming a win I have not booked. That is the next cut.

---

## The lesson

Token optimization is real, and it has a failure mode nobody warns you about: it can quietly cost you the intelligence you were paying for, with no error and no signal. A blanket "use the cheap model" rule. A default that thinks less. A file re-read so often it crowds out the work. None of these throw a warning. The agent just gets a little duller and you blame the model.

The fix was never "spend more." It was **spend where being wrong is expensive, save where it is trivial** — and verify the savings did not eat the quality. When something feels dumber, audit your own config before you blame the model behind it. Mine was the culprit every time.

---

## Related

- [The Health Check Was Reporting to a File Nobody Read](a-stub-a-dead-model-and-a-health-log.md): the same instinct not to blame the model, applied when the nightly learning loop silently stopped. A stubbed job, a retired model name, and a health log written where no reader looked.
- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — the earlier audit that found token burn in the automation layer; this one finds it in the context layer — same discipline, different surface
- [We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md) — the model-routing decision this post course-corrects: the right model is a per-task judgment, and both blanket-cheap and blanket-expensive are the mistake
- [Your Agents Are Only as Good as the Context You Program Them With](context-is-the-program.md) — the principle underneath the re-read fix: context is the program, and re-reading the same 3,700 tokens every turn was crowding out the work
- [The Boot Hook That Refired on Every Compaction](the-boot-hook-that-refired-on-compaction.md) — the same right-sizing principle applied to the context layer, plus the stale subagent model pins this post's routing fix eventually caught

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

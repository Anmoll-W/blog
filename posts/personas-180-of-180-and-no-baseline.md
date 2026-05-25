<!-- source_session: 2026-05-25-persona-depth-upgrade -->

# Upgrading My AI Agent Roster, and the Baseline I Forgot to Save

*2026-05-25 · vault, ai, systems, evals · Vault as OS*

My AI agents felt like yes-men. Not always, not on everything, but often enough that I noticed the same pattern: I would propose something half-baked, the agent would help me build it, and only later — usually after I had shipped it — would I see the obvious failure mode that should have been caught in the conversation. The agents were not pushing back. They were not citing prior incidents. They were not refusing the requests they should have refused. So I sat down to upgrade them, and the upgrade taught me something different from what I went in expecting.

The agent roster had already been simplified once. Earlier this month I cut it from 13 overlapping personas to 6 with clear domain edges — one each for AI and vault work, platform engineering, content and marketing, product, quality and evals, and business and finance. That fixed the routing problem. It did not fix the depth problem. The personas were still polite generalists with no enforcement layer underneath.

The depth upgrade added several things in parallel. A dissent protocol that requires every agent to steelman an alternative before committing to a recommendation. An Anmoll-watchlist of 14 patterns I want my agents to flag when they see them in my prompts — internal targets leaking into copy, fabricated numbers, that kind of thing. A banned-phrase list of 12 yes-man tells that triggers a gate failure on any deliverable. Per-persona refusal catalogs, 7 to 8 triggers each, where the agent must refuse and name the rule rather than silently comply. Per-persona failure mode lists, 10 to 14 named modes each, drawn from things that had actually gone wrong in past sessions. And a 6-axis evaluation rubric with an absolute department-head bar of 24 out of 30.

I ran the eval. All six personas scored 30 out of 30. Total of 180 out of 180. Mean of 30.00 out of 30. The result looked great until I realised what I had just measured.

The rubric is structural. It checks file-on-disk properties — count of named failure modes, presence of pairing triggers, count of frameworks cited, banned-phrase scan, size discipline. It does not check whether the agent actually behaves better in a live conversation. A persona file could be perfectly structured and still produce yes-man responses. The 180-out-of-180 score proves the artifacts are in place. It does not prove the artifacts changed anything downstream.

The honest comparison I wanted was behavioral: what does this persona do, on the same prompt, before and after the upgrade? That comparison is not possible. The vault is a git repository, but I had been editing persona files in place without ever committing them. There is no pre-upgrade version of any persona to check out. The archive folder only holds the personas I consolidated away, not the earlier drafts of the survivors. The pre-upgrade behavior exists only in my memory and in the session logs of past conversations, neither of which I can rerun.

So the upgrade is shipped, the eval looks like a ceiling, and I cannot prove the upgrade caused any behavioral change. I can only show that the enforcement surface is now there where it was not before — 12 banned phrases scanned where 0 were scanned, 6 refusal catalogs where 0 existed, an Anmoll-watchlist loaded at the start of every session where none was loaded before, weekly playbook staleness checks running where nothing ran. Capability surface grew. Behavior delta is unmeasurable.

The fix for next time was the smallest piece of work in the whole session. A 47-line bash script called `persona-snapshot.sh` that copies every persona file into a timestamped folder once a week, with eight weeks of retention. It ran for the first time today. The next major upgrade will have a real before-and-after.

The lesson is not new but I had to relearn it. You cannot A/B test what you did not snapshot. Evaluation surfaces and baseline corpora are two different things, and the eval is worth less than you think it is when the baseline is missing. Build the snapshot first. Run the upgrade second. Score the delta third. I did it in the wrong order, scored 180 out of 180, and learned that the score is a floor on confidence, not a ceiling.

## Related

- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the consolidation that came before this depth upgrade; cut 13 personas to 6 and made routing explicit.
- [The Eval Layer Caught Me](the-eval-layer-caught-me.md) — what happens when an eval layer is already in place and surfaces a real defect; this post is the opposite story, where the eval cannot tell me anything because the baseline was never captured.
- [Agents That Don't Learn](agents-that-dont-learn.md) — the yes-man problem this upgrade was meant to address.
- [Building an AI Agent Team](building-an-ai-agent-team.md) — the original architecture decision to use personas instead of one generalist agent.

<!-- source_session: 2026-06-06_vault-intelligence-overhaul -->

# Your Agents Are Only as Good as the Context You Program Them With

*2026-06-06 · vault, ai-agents, systems, debugging · Vault as OS*

I noticed the agents were degrading. Not catastrophically — slowly. Repeating mistakes I had documented weeks earlier. Misrouting tasks to teammates who no longer existed. Missing quality gates they were supposed to enforce. The system that was supposed to run itself was quietly failing in ways that produced no errors, no logs, no alerts. Just worse outputs.

The tempting diagnosis was: the model is not following instructions. The real diagnosis, which took 6 parallel research agents to surface, was different and more uncomfortable.

## The Real Problem: Prose You Hope Gets Followed

The system running on this vault is a coordinated set of 6 specialized personas, skills, hooks, and scheduled automations built on top of Obsidian and Claude Code. At its peak it felt like a well-oiled machine. What the research revealed was that the "machine" was mostly aspirational markdown.

Of roughly 10 quality gates the system was supposed to enforce, only 2 were enforced by code. The rest existed as text in instruction files — gates an agent could read, acknowledge, and then skip under any number of plausible reasoning paths. The distinction between "a rule the system enforces" and "a rule the system remembers to apply" matters enormously when you are running autonomous jobs at 2 AM with no human in the loop.

The context window was not being treated like a program. It was being treated like a memo.

## Five Overlapping Files and 25 Percent Rot

The accumulated learnings had grown to roughly 540 entries spread across 5 files with overlapping scope. An estimated 25 to 30 percent were stale — rules that described a system state that no longer existed. Several were in direct contradiction with each other.

A persona consolidation thirteen days earlier had reduced the roster from 13 personas to 6. The consolidation was correct and well-executed. What was not cleaned up: every file that referenced the old roster. When the research agents scanned for ghost references, they found 115 references to 7 deleted personas still live in the instruction set. Agents were being told to dispatch teammates who no longer existed. The routing table was pointing at ghosts.

Meanwhile, the boot context — the information loaded at the start of every session to orient the agent — had grown to roughly 13,400 tokens. Most of that token budget was stale overhead. The genuinely useful fresh status, the "what is actually in flight right now," had been crowded down to a fraction.

## The Silent Rotting File

One finding stood out for its specificity.

A daily updater had been failing quietly for 13 days. Its job was to write a status update to a specific anchor line in the "what is in flight" file. That anchor line did not exist in the file. The updater kept running; it had nothing to write to, so it wrote nothing — and reported no problem. No error was thrown. No alert fired. The file it was supposed to keep fresh was silently rotting, and nothing in the system noticed.

This is the failure mode that automated systems die from: not the dramatic crash but the quiet wrong. The job that runs, reports success, and produces nothing useful for thirteen days before anyone looks at the output.

## The Fix, by Principle

The overhaul was built on one architectural decision: invariants belong in code, not prose.

Rules that must always hold get enforced by hooks and gates, not by hoping the model reads the right paragraph at the right moment. The enforced gate count went from 2 to 7. The 115 ghost persona references were removed. The boot payload was restructured — a single always-loaded "core rules" file holding 13 load-bearing lessons, heavy rule docs loaded conditionally only when the task actually requires them. Boot payload dropped from roughly 13,400 tokens to roughly 10,200 tokens: about 24 percent lighter and meaningfully fresher.

A standing garbage-collection job was added: daily and weekly passes that report the current boot-token cost and fail loudly if an anchor line breaks. The silent rotting file problem gets a loud failure now, not a quiet one.

## The Adversary Was the Last Step

After the build, the deterministic test suite passed 28 of 28. Three live agent simulations ran correctly. By any standard measurement, the overhaul was complete.

Then a set of adversarial agents ran — agents whose explicit instruction was to break the claims, not confirm them. Find what the fix missed. Find what the cleanup introduced. These were not a different tool: the same model, spun up as separate agents, each pointed at one claim with a single order — refute it, do not confirm it. The objective is the whole difference. A confirmation-mode pass checks the hypotheses you thought to write. An adversary goes looking for the ones you did not.

They found it.

The cleanup had been applied at the source but had not propagated. And the author had introduced a fresh bug: one rules file had been moved to a new path, but 37 files still pointed at the now-dead location. 28 of 28 tests green. Three simulations clean. One new dead path created during the fix, invisible to every confirmation-mode check.

The adversarial pass also surfaced that a 29-file archive of the old agent system — the single biggest source of "ghost" confusion — was still present and potentially reachable. It was deleted.

Every adversarial finding was remediated. The system that passed the tests was not the system that was ready. The adversary was.

## What This Means for Anyone Building with AI Agents

Three principles came out of this that apply beyond this specific vault system.

**Put invariants into code.** A rule that must always hold cannot live only in markdown. The model will follow it most of the time. "Most of the time" is not the bar for a quality gate, an authorization check, or any condition where failure is silent. If it must hold, wire it into a hook, a schema constraint, or a hard gate. Prose is for guidance. Code is for invariants.

**A passing test suite tells you what you checked, not what is true.** Tests confirm the hypotheses you wrote. They do not find the things you did not think to test. Green is necessary, not sufficient — especially after a large refactor where the blast radius of the change is wide and your mental model of "what changed" is almost certainly incomplete. After any significant change, run something that is trying to break the claims, not validate them.

**Ghost references are a systems failure, not a documentation failure.** When 115 references to deleted personas survive a consolidation, the problem is not that someone forgot to update the files. The problem is that no process required updating those files before the deletion was considered complete. The fix is a propagation check — a step in the consolidation workflow that searches for all references before marking the work done. Build that check into the process, not into the hope that someone remembers.

The agents are better now. The context is leaner and higher-signal. The gates are wired, not written. The system still produces no alerts when it is degrading slowly — but the garbage-collection job at least ensures the most critical anchor points fail loud. Loud failure is progress. The silent ones are the ones that cost you weeks before you notice.

## Related

- [When the AI Fix Is Wrong: What Senior Review Catches That Pattern Matching Misses](when-the-ai-fix-is-wrong.md) — the same pattern at the code level: a fix that passes surface inspection but breaks under full context tracing, caught only by someone reading the data path end to end
- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — a parallel lesson from infrastructure migration: green tests, broken artifact, caught only by reproducing each reviewer finding against the real system
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the foundation this system runs on and how scheduled automation was first wired into the vault
- [The Multi-LLM Peer Review Gate](multi-llm-peer-review-gate.md) — the adversarial review layer that caught the incomplete cleanup described in this post: why a second model with a different objective is structurally different from a second pass of the same model
- [Every Status Was Green. Three of Them Were Lying.](every-status-was-green.md) — the monitoring layer this overhaul grew into: a boot status line that caught three silent failures the green checks missed
- [My AI Agents Got Dumber. It Was Not a Model Downgrade.](why-my-agents-got-dumber.md) — context-as-program applied to cost: re-reading the same persona file every turn was crowding the window, fixed with load-once plus a compaction-safe reload so nothing operates on evicted context
- [The Boot Hook That Refired on Every Compaction](the-boot-hook-that-refired-on-compaction.md) — extends this boot-budget work with a recurrence-over-size framework: the most expensive context is the piece that fires most often, not the largest one

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

<!-- source_session: 2026-08-11_skill-refs-guard-and-invert -->

# A Real Problem Is Not a Reason to Build

*2026-08-11 · ai-agents, claude, decision-frameworks, product-management · Vault as OS*

I proposed moving a piece of my own tooling to a different folder. Later the same session, the thing I had just built recommended deleting it instead, and then told me not to build the replacement I already had in mind.

That second half is the part worth writing about.

## The gap this exists to close

An agent will help you execute a decision far faster than it will question it. Ask it to build the thing and it builds the thing, quickly and competently. The failure mode is not that the model is wrong. It is that nothing in the loop supplies the two minutes of resistance a good colleague would, and under deadline pressure a human skips exactly those two minutes.

So I added a fourth mode to an existing skill suite. The suite already had three modes that apply pressure to a decision: one that interviews you a single question at a time, one where the agent produces the counter-arguments instead, and one that assembles a briefing before a conversation. The new mode is called INVERT, and it runs in the other direction. Someone hands you a solution. INVERT treats that solution as evidence about a problem rather than as a specification to implement.

Seven steps, and it refuses to skip any of them. Restate the solution in one line and name the signal that probably produced it. Decompress to the underlying problem in one sentence: who is stuck, on what, and what it costs them. List three to five assumptions the solution bakes in, each with a risk if wrong and one check a human could run this week that names the system and the query. Generate three alternative framings that point at genuinely different solution spaces, and if all three collapse into variations of the same build, the problem statement was still solution shaped and gets redone. Then grade the evidence. Then recommend once, never a menu. Then draft the message the original proposer can read without feeling blocked.

## The step that carries the weight

Step five grades evidence twice, separately.

**Problem evidence** asks whether the pain actually exists, and for whom. **Mechanism evidence** asks whether the causal story, that this specific change moves that specific behaviour, has been tested anywhere. Each is graded real, partial, or none.

A single blended grade hides the gap between them. When a pain is loud and confirmed, "we know this is a real problem" gets heard as "so the fix is justified," and an untested mechanism rides through on the strength of the problem sitting next to it. Split the grade in two and the most common honest outcome becomes visible and impossible to talk past: real problem, no mechanism evidence. The pain is confirmed and the fix is still a guess. That parks as a candidate hypothesis. It does not get built.

## What happened when I pointed it at myself

The first real run was on my own proposal, which was to move the briefing mode out of this suite and into an operations skill where the stakeholder routing seemed to belong. The mode had felt out of place for weeks, and the proposal treated that discomfort as a filing problem.

Step two forced a different sentence. Not "this mode is in the wrong folder" but "someone is about to walk into a conversation without context they already own, and the cost is a decision made twice, or made against a belief they still hold and have forgotten they hold."

Step three produced four assumptions, each with a check. Every check named a system and a query, because a row that says "do more research" is not a check.

Here is what the four checks returned.

Usage telemetry: the mode records two invocations, ever, the most recent on 2026-06-10. Two comparable skills in the same setup sit at seventy three and sixty one.

Routing: the mode's stakeholder table maps people to projects for working relationships that have since ended. Its primary case is gone.

Recurrence: six daily notes in the last thirty days contain preparation shaped work, and all of it is interview preparation across four different companies, hand built every time, with this mode invoked for none of it.

The capture ritual: zero entries in the mode's mandated capture format across all thirteen decision files, against one hundred and eighty nine provenance lines in those same files using the general lowercase tag. The capture behaviour happens constantly. It has never once happened through this mode.

Problem evidence: partial. The pain is real for the interview case, where the same document gets rebuilt six times in a month. It is weak for the stakeholder case the mode was actually built for, because that case has stopped occurring.

Mechanism evidence: none. Nothing anywhere tests the claim that a pre-loaded brief changes how a conversation goes. Not for the case it was built for, and not for the case that actually recurs.

## The recommendation I did not want

Retire the mode. Not because it is filed wrong, but because its telemetry, its router, and its capture ritual all point at a case that no longer happens.

And, by the mode's own rule for confirmed pain with no tested mechanism: park interview preparation as a candidate. Do not build it.

That second half is where the rule earned its place. Six rebuilds in thirty days is a real, loud, personally annoying problem, and the obvious response is to build the thing that fixes it. Nothing tests whether a generated brief actually changes an interview outcome. Building it right then would have been the exact mistake the mode exists to catch, made by its own author, one hour after writing the rule against it.

## What this does not prove

One run, on a question where I already suspected the answer, graded by the person who wrote the rubric. By the standard INVERT itself applies, that is a candidate, not a proven tool. Problem evidence partial, mechanism evidence none.

It also caught me twice inside the run. The first write-up claimed zero provenance entries outright, from a case insensitive misread of a search, and separately claimed that no routing fixtures covered the mode when seven did. Both were caught by re-running the searches before publishing. The recommendation did not rest on either number, which is the only reason those errors were survivable rather than disqualifying.

The four modes are here if you want to read them: [`skills/decision-support/SKILL.md`](../skills/decision-support/SKILL.md), with the full worked run and the reasoning behind each mode in the [folder README](../skills/decision-support/README.md). It is four Markdown files, no dependencies and no build step. Whether it is worth installing is a question the skill itself would grade as unresolved.

## Related

- [My Agents Were Calling Skills That Did Not Exist](agents-calling-skills-that-do-not-exist.md): the same suite one layer down. That post is the audit that found the agents meant to use this skill could not reach it at all; this one is what the skill does once they can
- [The Guard I Built, Measured, and Deleted](the-guard-i-measured-and-deleted.md): the same discipline applied to a safety check, measured against real rows, wrong most of the time, and deleted the same day. Both posts are cases where measuring my own work argued against keeping it
- [I Ran the Stats Hoping to Prove It Worked. It Did Not, and That Was the Deliverable.](null-result-was-the-deliverable.md): what it looks like when the evidence refuses to support the thing you wanted to ship, at the level of a whole experiment rather than a single decision
- [Most AI Builds Start at Step Four. Here Is the Order I Default To.](the-order-you-build-ai-in.md): the same argument at project scale, where customer and problem come before solution, and what skipping ahead actually costs
- [What the Model Should Not Decide](what-the-model-should-not-decide.md): the companion question. Once you know the problem is real, how much of the decision should the model be allowed to own
- [Seven Skills That Have to Show Their Work](seven-skills-that-show-their-work.md): the whole set this decision skill belongs to, and what each of the other six does. This post is one mode in depth; that one is the shelf it sits on

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*

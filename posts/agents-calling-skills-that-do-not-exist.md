<!-- source_session: 2026-08-11_skill-refs-guard-and-invert -->

# My Agents Were Calling Skills That Did Not Exist

*2026-08-11 · ai-agents, claude, skills, tooling · Vault as OS*

I saw a product manager post a skill they had written called `/problem-first`. The idea is simple and good: when somebody hands you a solution, reverse it back into the problem before you agree to build anything. I wanted that mechanic in my own agent stack, so I added it as a fourth mode inside a skill suite I already had, rather than creating a new skill for it.

Building it went the way building a prompt goes. Then I ran one test that turned a small afternoon into a much longer one.

## The test I almost did not run

The test was this: give an agent a task shaped exactly like the new mode, tell it nothing about which skill to use, and watch what it picks.

It never picked the new one. Not once.

My first theory was wording. The description field is what an agent reads when it decides whether to load a skill, so I rewrote it: triggering conditions first, no summary of the steps, because a description that summarises the workflow tends to get followed instead of the skill body. That was a real improvement, and it was not the binding constraint.

The binding constraint was that subagents receive skill names without descriptions at all. No amount of rewriting fixes a field that is never delivered. Wording was the thing I could see, so it was the thing I fixed first.

## The question I had never asked about anything else

Underneath the wording problem sat a plainer question. Can the agent reach this skill at all?

I had never asked that about any of the others. So I wrote a checker. It walks every persona file and every agent definition, pulls out every skill name they declare, and tries to resolve each one against the skills actually installed on disk.

Forty-one names resolved to nothing.

None of them were typos. They were the residue of a consolidation on the third of July, when I merged a sprawling skill catalogue down into a small number of suites. The suites were correct. The files pointing at the old names were never rewritten. Every agent carrying one of those references had been sent hunting for something uninvocable, quietly, for weeks, and nothing anywhere reported it.

## The checker was wrong twice

Before that number is worth anything: the checker itself was wrong twice, and both times it was wrong in the direction that feels productive.

The first version used a greedy pattern to pull the skill name out of a list item, so it grabbed the last backticked word on the line instead of the first. A bullet documenting a retirement, naming the dead skill it had replaced, got reported as declaring that dead skill.

The second version enumerated two of the three places a skill can live on my machine. It missed a directory of single-file skills entirely and reported those as broken. With that fixed, the count of installed skills went from seventy-four to one hundred and two.

A checker that manufactures its own findings is worse than no checker, because you act on the output. I only caught both because I planted a fake broken reference, watched the checker fail, removed it, and watched it pass. If you have not seen your check fail on purpose, you do not know it works.

## The fourth place, and the one with teeth

Three declaration sites, then. Persona files, agent definitions, and the skill files themselves.

There is a fourth, and it is the one that actually moves traffic. A hook on my machine scores every prompt against a table of patterns and injects an instruction naming the skill to load. My checker never looked at that table. When I pointed it there, three of the fourteen targets in the table resolved to nothing.

That site matters more than the other three. A dead reference in a persona file is a suggestion an agent can ignore. A dead row in the routing table is an instruction, injected into every matching prompt, sending the agent after something that cannot be loaded.

## Deleting all three would have been the confident move

One of the three was an operations skill for a product I had handed off. I deleted that skill on the nineteenth of July and left its three routes behind. They had been dead for twenty-three days. Removing them was the easy call.

The other two had no trace in version history and no directory anywhere on disk, which is exactly what the first one looked like. So I deleted those too, and the routing test suite went from passing to twenty-nine failures.

That is what sent me to look properly. My configuration file keeps a usage record for each skill. One of the two names I had just deleted showed seventy-four invocations, the most recent on the twentieth of July. The other showed fourteen, the most recent on the eighth of August, three days before I deleted its route.

Both skills are genuinely gone from disk. But a skill that ran fourteen times, most recently last week, is not debt. It is something that went missing. Deleting its route would have made the loss permanent and silent, and the test run would have gone green while doing it.

I put both routes back, restored their fixtures, and wrote the two names into the checker's baseline file with a note saying exactly why they are there and the condition under which those lines should be deleted. The check is now honest about everything except two things it is deliberately loud about.

The lesson is narrow and I keep relearning it. Absence on disk is a proxy. Invocation count is the outcome. When the two disagree, the outcome wins.

## Then I pointed the new mode at the thing that started this

Once the plumbing was fixed, I ran the new mode on my own suite, on the question of whether one of its older modes still earns its place. That mode prepares a briefing before a meeting with a specific person.

It has been invoked twice. Ever. The last time was the tenth of June. Two comparable skills in the same stack sit at seventy-three and sixty-one.

The routing table that mode depends on maps people to projects, and those working relationships have ended. It also specifies a capture ritual that writes notes in a particular format after a meeting. Nothing anywhere on my machine is written in that format.

Meanwhile, interview preparation appears in six daily notes over the last thirty days, across four different companies, hand-built every time, because no mode covers it.

So the recommendation was to retire the thing I had been about to improve, and to leave the interview idea unbuilt and parked, because the problem is real and the fix is still a guess. I did not expect the first serious run of a new tool to tell me to delete something.

## What I do not know yet

That is one run.

One run is not evidence that the mode works. It is evidence that the mode produced a defensible answer once, on a question where I already suspected the answer, graded by the person who wrote it. All three of those weaken it. The honest status is unproven, and it stays unproven until it has been pointed at decisions where I do not already know what I want to hear.

What I can say with numbers is duller and narrower. Forty-one dead references in the files that describe my agents. Three more in the table that routes them, two of which turned out to be losses rather than debt. And a checker that now fails when any of it comes back.

## The skill

The suite lives in this repository at [`skills/decision-support/SKILL.md`](../skills/decision-support/SKILL.md), with its longer mode references in the folder beside it. It is the file that runs on my machine, with the private routing details split out into a separate file that stays local. Copy the folder into your own skills directory if you want to try it.

Take it as a working artifact rather than a recommendation. The section above says what I do not know about it.

## Related

- [The Guard I Built, Measured, and Deleted](the-guard-i-measured-and-deleted.md): the other half of the same discipline, where measuring a guard I had already written was what proved it should not exist. This post is the case where the measurement said keep it, and changed what it was checking
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md): where the skills layer described here was originally wired to the personas. That post assumes the references resolve; this one is what happened when I finally checked
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md): the earlier version of the same discovery, at the persona layer instead of the skill layer. Built, documented, wired together, and not actually reaching anything
- [Every Status Was Green. Three of Them Were Lying.](every-status-was-green.md): the same failure shape one level up, where a passing result stands in for a check that never ran. Deleting those two routes would have produced exactly that
- [What I Learned Auditing an AI Agent Repository Built by Another Product Manager](auditing-another-pms-agent-repo.md): the last time somebody else's agent work sent me back to my own. That audit started with their repository; this one started with their skill

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*

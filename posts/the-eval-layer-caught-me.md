<!-- source_session: 2026-04-27_karpathy-discipline-rollout -->

# The Eval Layer Caught Me Violating My Own Rules

*2026-04-27 · ai, vault, claude-code, discipline, eval · Vault as OS*

A tweet from Andrej Karpathy listed five ways AI coding agents fail. I recognized myself in every one. So I sat down to install discipline against them in my CLAUDE.md files. My own eval agent caught me violating the principles I was installing.

This post is about what changed and why having an eval layer is what made the discipline stick.

## What the tweet actually said

Karpathy named the patterns AI agents default to when writing code. They pick an interpretation silently when the task is ambiguous. They overcomplicate simple work. They drift into adjacent code that "needed cleanup." They drop comments they did not understand.

I had felt all of these. I had also written rules against them — vague ones. "Do less. Verify more. Push back with data." Real intent, not sharp enough.

The principle that mattered most was the third one. Surgical changes. Touch only what the task requires. Do not improve adjacent code. Match existing style even if you would do it differently. Every changed line traces to the user request.

I had no rule like this anywhere in my system. The closest was "small focused diffs," which is a different idea. Small diffs can still touch the wrong files. Surgical means scope, not size.

## The cascade

My CLAUDE.md files exist at three levels. A global file at `~/.claude/CLAUDE.md` that loads in every session. A workspace file that loads when the project map is the active root. And a project-level file in each codebase or vault project where the architecture and tripwires live.

I added the four principles at the global level first. Surgical changes, simplicity first, think before coding, tests as goals. Sixteen lines. The principle frame was clean.

Then I extended it. Workspace-level: project-specific tripwires pulled from past-mistakes. Project-level: editing rules for each codebase and content folder. Across thirty files.

That is where I drifted.

## Where I drifted

I added the same four-bullet block to a twenty-two-line Finance file. Same to a thirty-five-line Travel file. Same to an eighteen-line Health file. The bullets were not wrong. They were already in the global file. I was repeating myself across files that *already loaded the global*.

The decision I had not named: do I add discipline by cascade — same rules at every level — or by gradient — global rules everywhere, project-specific rules only where they introduce a new constraint? I picked cascade by default and only realized after thirty files were edited.

The signal: every product manager has had a developer touch unrelated code "while they were in there" and break something. The fix is not to write a longer style guide. It is to name the principle and put it where everyone already reads it. Once. Not in twenty places.

## The eval layer

Vera is my eval agent. Her job is not to write things. Her job is to stress-test what I have written before it ships.

She runs a fixed protocol. Read the relevant rubric first. Score against each criterion. Output in three parts: ranked fixes by impact, what is already solid, one blind-spot question. Mandatory output block at the end, no exceptions.

I asked her to review the rollout. Her top three findings:

One. Naming inconsistency. The global file said "Coding Discipline." The workspace file said "Editing Rules." The project files said "Editing Discipline." Same content, three header names. A future session could not grep one term and find them all.

Two. Two files in the workspace had identical content. One was supposed to describe a UI prototype. The other a different internal dashboard. Someone had copy-pasted at some point and the duplication had survived for months. The discipline rollout could not be marked complete on these two until the duplication was resolved.

Three. The Tier B bloat I described above. Three out of fourteen content files received additions where the global file already covered the rule. Recommendation: trim to a single pointer line. Keep specific bullets only where they introduce a new project-specific rule.

She also told me what was already solid. The four-principle frame survived all twenty-six edits without dilution. Every project tripwire traced to a real entry in past-mistakes — no aspirational rules. Verification was framed concretely on every project file. Eight active automations were healthy.

Then the blind-spot question: what happens when global, workspace, and project-level rules conflict? Does the system apply them additively, or does the last-loaded rule silently override the earlier ones? No precedence test exists.

That is the question I would have missed.

The objection to an eval layer is overhead — another agent, another step before shipping. The cost of an eval layer is bounded: Vera runs a fixed protocol against a fixed rubric and outputs a fixed shape. The cost of not having one is unbounded. Every drift compounds. A system that drifts long enough is one that eventually disagrees with itself.

## What I Learned

**Technical:** discipline scales differently than rules. Rules go in many places — they are local context. Discipline goes in one place where everyone reads it, and project-specific *tripwires* go where they add a new constraint. Confusing the two produces bloat that violates the principle of simplicity it was supposed to encode.

**Decision-making:** every system that makes decisions needs an eval layer that catches its own drift. Karpathy named the patterns for AI agents. The same patterns hit human product managers daily — pick an interpretation silently, overcomplicate, drift into adjacent scope, ship without verifying. The fix is not better rules. It is a separate agent whose only job is to stress-test the work before it ships, with a fixed protocol, citing best practices, never approving because someone seems confident.

That agent caught me violating the principles I was installing. That is the whole point.

---

**[2026-04-27](../README.md#all-posts)** · [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![vault](https://img.shields.io/badge/vault-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![claude-code](https://img.shields.io/badge/claude--code-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![discipline](https://img.shields.io/badge/discipline-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![eval](https://img.shields.io/badge/eval-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Vault as OS →](../series/vault-as-os.md)

## Related

- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — Vera's onboarding and why an evaluating agent has to come from a different posture than a producing one
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automated system this discipline sits inside
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the same eval posture applied to session infrastructure
- [My AI Agents Were Running. They Just Weren't Working.](self-correcting-agents.md) — discipline rules without an eval layer drift the same way agents do without one
- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the architecture Vera sits on top of

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

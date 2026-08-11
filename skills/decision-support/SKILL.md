---
name: decision-support
description: Use when a decision needs pressure before it ships. Four situations. (1) Someone hands over a solution, feature request, or roadmap item and it is not yet established what problem it solves, or an idea backlog needs triage. Triggers: "invert this", "what's the real problem", "why do we think we need X", "is this the right thing to build", "should we build this", "the team wants X", and any request to scope or spec a solution someone else proposed. (2) Stress-testing a plan the user will defend: "grill me", "stress-test this", "poke holes", "interview me about X". (3) Attacking a decision: "devil's advocate", "pre-mortem", "red team", "test my assumptions", "what could go wrong". (4) Preparing for a person: "prep for X", "meeting prep", "before the call", "/prep".
---

# Decision Support

Four modes. State the mode first, in one line, before anything else.

**Environment layer.** If `references/local-context.md` exists, read it before any mode
that needs a person, a project, a data system, or a file path. It supplies all of them.
If it is absent, this skill still runs: ask one question for the routing you need, use
the user's own named systems, and say "no local context file, using what you gave me".
Never invent a stakeholder, a path, or a data source.

**Mode bodies.** `references/` holds the full originals for PREP (`prep.md`), GRILL
(`grill-me.md`), and CHALLENGE (`the-fool.md`). Read the mode's file when it is present.
When it is absent, the summary below is the complete instruction set, not a pointer:
execute it as written. INVERT has no reference file by design and is fully inline.

## Choosing the mode

Each row is self-sufficient. Match on what the user wants to happen, not on the phrase
alone, because the phrases overlap.

| You want | Phrases | Mode |
|---|---|---|
| A pre-read on a person or project before a conversation. Nobody is challenged yet. | "prep for X", "meeting prep", "before the call", "/prep" | **PREP** |
| To be interviewed. **You answer the questions**, one at a time, about a plan you will defend. | "grill me", "interview me", "stress-test this **plan**" | **GRILL** |
| **I produce the analysis.** Counter-arguments, failure narratives, evidence audit, no questions back. | "devil's advocate", "pre-mortem", "red team", "test my assumptions", "what could go wrong", "poke holes", "stress-test this **artifact**" | **CHALLENGE** |
| A solution is already on the table and the question is whether it aims at the right problem. | "invert this", "what's the real problem", "why do we need X", "should we build this", "the team wants X", "triage my idea backlog" | **INVERT** |

Tiebreak, in this order: (1) Is a solution already proposed and unexamined? INVERT.
(2) Who does the talking? User answers means GRILL, I answer means CHALLENGE.
(3) Is anything being challenged at all? No means PREP.
Say which tiebreak you used when the request was ambiguous. Offer a second mode at the
end when it would add value.

## PREP mode, stakeholder pre-read

1. **Route.** Use the stakeholder table in `references/local-context.md`. No file, or no
   match: ask ONE question ("Who is the conversation with, and which project?"). State
   `Routing to: [Project], [role]`.
2. **Load the context layers in parallel** from the paths in `references/local-context.md`:
   decision log (last 3 in 30 days plus provenance tags), hypothesis list (all proposed or
   candidate: claim, confidence, next test), today's task list (open items for this
   project, max 5), session log (last 3 entries). Missing file: "Not found, context layer
   skipped". No local context file at all: ask the user which files to read, or work from
   what they paste, and label the brief `partial context`.
3. **Conflict surface, the core value step.** For each live hypothesis, does a recent
   decision act against it? Patterns: claims X is the bottleneck but a decision divested
   from X; proposes test Y but a decision locked against Y; high confidence but the
   decision was made without resolving it. Output a `CONFLICT:` block with a recommend
   line, or the sentence "No hypothesis-decision conflicts detected."
4. **Brief**, under 300 words: Recent Decisions, Open Hypotheses, Open Tasks, Last
   Session, Conflicts, Suggested focus (1 to 3 bullets derived ONLY from loaded context).
   Never fabricate. Overdue means next-test date earlier than today.
5. **Post-meeting capture, required ritual, never optional.** On "done" or "meeting
   wrapped", ask the 3 capture questions (verbal commitments, new evidence, new
   hypotheses or questions) and route each: a verbal commitment to the decision log with
   a verbal provenance tag and the date; evidence to the matching hypothesis; a new
   hypothesis as `status: candidate`, `confidence: low`; a contradiction to counter
   evidence plus a downgrade check. Confirm with a 3-line `Captured:` block.

Reads only, except the capture loop.

## GRILL mode, the relentless interview

Mindset: the user's CTO who cares whether this works in three months. Skeptical, curious,
charitable. Honesty over agreement.

1. **Anchor first.** Restate the plan in 2 to 4 bullets. Do not question until confirmed.
2. **Build the decision tree silently:** problem framing, solution choice versus rejected
   alternatives, scope and smallest proving slice, load-bearing assumptions, failure
   modes, reversibility and blast radius, execution critical path, verification metric
   and disconfirming evidence, three-month regret. Pick the most under-examined branches
   for THIS plan.
3. **ONE question per turn, the core rule.** No "and also". Concrete, answerable, targets
   one branch, builds on the last answer.
4. **Track the ledger** (Resolved, Open, Unraised). Surface it every 4 to 6 questions.
5. **Push back with reasoning, not opinion.** Name the specific weakness, offer the
   counter-scenario, cite the user's own prior statements. On a concession: mark it open,
   move on, no piling on. Catch every "we'll figure that out later" and put it on the
   open list explicitly.
6. **Stop when** all load-bearing branches are resolved and the user can play the plan
   back with no hand-waving, or the user decides to rework, or the user taps out.
   Frustration alone is not a stop signal: on genuine stuckness offer "keep pushing, or
   park this branch?"
7. **Close with the written artifact:** `## Plan` one-liner, `### Resolved`,
   `### Still open` (why each matters), `### Recommended next steps` (most load-bearing
   open branch first).

Never: multi-question turns, Socratic theater, generic questions ("have you considered
edge cases?", name the edge case), performative skepticism, rudeness.

## CHALLENGE mode, structured critical reasoning

1. **Steelman first.** Restate the position in its strongest form. Confirm before attacking.
2. **Pick the reasoning mode with the user, never for them.** Where an interactive
   question tool exists, use it. Where the harness has none, state the recommended mode
   and the one-line reason, then proceed unless corrected. Map: question assumptions to
   Socratic or Falsification; find weaknesses to Pre-mortem or Red team; build
   counter-arguments to Dialectic. Recommend by context: untested beliefs to Socratic, a
   plan about to be committed to Pre-mortem, adversarial exposure or a stranger using it
   to Red team, a data-backed claim to Falsification, a strategy debate to Dialectic.
3. **Challenge.** Present the 3 to 5 STRONGEST points only, depth over breadth, each
   grounded in specific concrete reasoning. No vague what-ifs.
4. **Engage.** The user responds before synthesis.
5. **Synthesize.** A strengthened position integrating what held up. Concede honestly.
   Never leave a pile of objections without synthesis. Offer a second pass in a different
   reasoning mode.

Outputs by reasoning mode: Socratic gives an assumption inventory, probing questions by
theme, suggested experiments. Dialectic gives thesis, antithesis, synthesis, confidence.
Pre-mortem gives ranked failure narratives, early warnings, mitigations, inversion check.
Red team gives adversary profiles, ranked attack vectors, perverse incentives, defenses.
Falsification gives claims, falsification criteria, evidence grades, competing explanations.

Never: strawman, disagreement for its own sake, nihilism, minor-objection stacking,
overriding domain expertise with generic skepticism.

## INVERT mode, solution to problem reversal

Use when someone hands over a solution ("we need X", "let's build Y") and the underlying
problem needs checking before committing, or when triaging a backlog of your own ideas.
Two directions, same 7 steps: forward for someone else's proposed solution, reverse for
your own idea fed in the same way.

1. **Take the solution as evidence, not a spec.** Restate it in one line. Name the likely
   signal that produced it: a support pattern, a competitor move, a personal frustration.
   If a project is named and you have file access, read its decision log, hypothesis list,
   and support-data file (paths in `references/local-context.md`) before writing anything.
   A prior finding on this exact question outranks fresh reasoning. If you cannot read
   them, say so in one line and continue. Never state what those files say without having
   read them.
2. **Decompress to the underlying problem.** One sentence: who is stuck, on what, and what
   it costs them (trust, time, money, retention). Ground it in real data from a system the
   environment actually has. No data found: label the problem statement `unverified` and
   carry that label through steps 5 and 6. Still complete every step. Never let an
   unverified problem be recommended as if it were confirmed.
3. **List 3 to 5 assumptions baked into the proposed solution.** Each with the risk if it
   is wrong, and one check a human could run this week, naming the system and the query
   ("support tool: tickets tagged billing, last 50", "analytics: exit rate on the pricing
   page, 28 days"). Never "do more research". These rows are recommended actions, not
   results: never invent a ticket ID, a count, or a finding to make a row look concrete.
4. **Generate 3 alternative framings of the same underlying problem**, each pointing to a
   genuinely different solution space. If all 3 collapse to variations of the same build,
   the problem statement in step 2 is still too solution-shaped: redo step 2 once. If the
   second attempt still collapses, stop and say so. A problem that genuinely admits one
   solution space is itself a finding. Never invent a third framing to fill the slot.
5. **Grade evidence twice, separately, because a single grade hides the gap.**
   **Problem evidence:** does the pain actually exist, and for whom. **Mechanism
   evidence:** has the proposed solution's causal story (this change moves that
   behaviour) been tested anywhere. Each is `real`, `partial`, or `none`. Read the pair:
   - Problem `none`: kill it, or convert it into a research task. Never a build, whatever
     the mechanism grade. A tested mechanism aimed at a problem nobody has is still nothing.
   - Problem `real` or `partial`, mechanism `none`: the pain is confirmed and the fix is
     still a guess. Park it as a candidate hypothesis with the test that would settle it.
     Do not build.
   - Problem `real` or `partial`, mechanism `partial`: build only the smallest slice that
     would falsify the causal story, not the full solution.
   - Problem `real`, mechanism `real`: build it, and name the metric that would tell you
     the mechanism failed in this specific context.

   State both grades in words. Never blend them into one verdict, and never imply one.
6. **Recommend.** Pick one framing, or explicitly kill the idea, and say why in one line,
   tied to the evidence pair from step 5.
7. **Draft message.** 2 to 3 sentences the original proposer can read without feeling
   blocked: what problem you are chasing on their behalf, what you found, and what you
   are proposing instead, or a confirmation that they were right.

Never: skip the assumption-risk table or the alternative framings under time pressure,
that is the exact failure mode this mode exists to prevent. Never accept 3 framings that
are cosmetic variants of one build. Never let this mode produce a template library or a
menu of solutions to pick from: its output is always one recommendation plus the
reasoning behind it.

## Verification

- **Every mode:** runs to completion with `references/local-context.md` absent. Any step
  that needed it said so instead of inventing a name, a path, or a data source.
- **PREP:** brief under 300 words, every context layer cited or marked skipped, conflict
  check explicitly run, capture loop fired after the meeting or explicitly declined.
- **GRILL:** final artifact produced (Resolved, Still open, Next steps), and every turn
  carried exactly one question.
- **CHALLENGE:** thesis steelmanned before the challenge, reasoning mode chosen with the
  user or explicitly recommended, 3 to 5 challenges maximum, synthesis section present.
- **INVERT:** the assumption table names a system and a query per row, the 3 framings are
  genuinely distinct solution spaces or step 4's stop clause was invoked, BOTH evidence
  grades stated in words, one recommendation only and never a menu.
- Append a one-line Changelog entry after meaningful use.

## Changelog
- [2026-07-03] Suite created. Absorbed prep (vault playbook body copied to references/prep.md), grill-me, the-fool (full bodies in references/). Stricter rules kept: prep's mandatory post-meeting capture; grill-me's one-question rule; the-fool's steelman plus synthesis contract.
- [2026-08-11] Added INVERT mode (solution-to-problem reversal plus evidence-status triage), prompted by an external PM's `/problem-first` skill seen on X. Built as a 4th mode here rather than a new top-level skill or a public product: it is a forcing function (rigor a PM would skip under pressure), not a template, so it fits the suite's existing judgment-pressure job. Internal use only for now; promote only if it proves out on real decisions. See `Knowledge/decisions.md` 2026-08-11 entry.
- [2026-08-11] Discoverability pass after a 3-arm test showed a free-choice agent never selected this skill for INVERT-shaped work. Description rewritten so the solution-handed-over triggers lead (773 chars, no workflow summary, since a description that summarises steps gets followed instead of the body); INVERT route added to `hooks/conditional-rules.sh` SKILL_MAP with 9 new regression fixtures (7 HIT, 2 over-breadth MISS guards), verified by executing the hook itself and not only the pattern extractor; 11 stale `the-fool`/`grill-me` persona refs plus 3 in `pg-startup-eval` repointed here with the mode named; INVERT steps 1, 3 and 5 tightened. Known limit: subagents receive skill names WITHOUT descriptions, so subagent discovery cannot be fixed by wording; persona files are the reachability path there.
- [2026-08-11] Vera stress test for robustness and multi-user portability. Nine defects fixed. (1) Removed a fabricated frequency claim ("the most common honest outcome is real problem plus none mechanism") written the same day with zero INVERT runs behind it; it also anchored the verdict before the evidence was graded. (2) Three published triggers did not route, proven by executing the hook: "test my assumptions", "what could go wrong", "interview me about X"; added to SKILL_MAP with fixtures. (3) The mode table matched on phrases and sat above its own tiebreaker, so "stress test this skill" resolved to GRILL when CHALLENGE was correct; rows are now self-disambiguating and an explicit 3-step tiebreak follows. (4) Two-layer portability split: every stakeholder name, project name, data system and filesystem path moved to `references/local-context.md`, which SKILL.md reads only if present, so the skill runs standalone in any environment and carries no private names. (5) PREP and GRILL had no absent-reference-file fallback where CHALLENGE did; all three now state that the inline summary is the complete instruction set when the file is missing. (6) CHALLENGE step 2 hard-depended on an interactive question tool that does not exist in every harness; now degrades to a stated recommendation. (7) Step 4's "redo it" loop had no iteration cap; capped at one retry with an explicit stop clause, since a single-solution-space problem is a finding rather than a failure. (8) Step 5 defined 3 of 9 evidence-grade combinations; the pair is now read exhaustively, including the previously undefined case where a tested mechanism aims at a problem nobody has. (9) Removed a dangling pointer to "the original pack" and brand idiom meaningless outside this environment. Open fork, deliberately not decided: whether PREP belongs in this suite at all, given it applies no judgment pressure.
- [2026-08-11] Agent-reachability pass, the third channel. The description reaches the main session, the `SKILL_MAP` hook reaches user prompts, and neither reaches a subagent, which receives skill names with no descriptions at all. Persona and agent-definition files are the only path in, so: INVERT added to `maya.md` (a content format proposed as the answer with no audience problem named) and `vera.md` (a solution handed to the eval gate before anyone checked it solves the right problem), the two personas that carried the suite but not the mode. `~/.claude/agents/code-review.md` and `qa-check.md` were still invoking `the-fool`, retired 2026-07-03, `qa-check` on every single run; both repointed with the mode named, and `qa-check` told what to do at CHALLENGE step 2, which normally asks the user to pick a reasoning mode and has no user to ask inside a subagent. `draft-post.md` deliberately got nothing: it is fire-and-forget drafting, and a skill reference it should not act on is worse than none. Two-layer split documented in `Knowledge/skills-registry.md`, not only in `decisions.md`. New guard `hooks/tests/retired-skill-refs.test.sh` derives retired names from the registry's own REMOVED tombstone rows, so retiring a skill arms the check with no second list to drift; it and the privacy guard now run daily as invariant F of `vault-runners/preflight-guard.sh`, because both had been scripts nothing scheduled ever ran. Verified: 180 route fixtures pass, both guards exit 0 clean and exit 1 with the file and line named when the real bug is replanted.
- [2026-08-11] First INVERT run on a real decision, dogfooding the mode on the suite's own open fork: does PREP belong in this suite. Evidence pulled live, not recalled. (1) Skill telemetry: the standalone `prep` skill records 2 invocations ever, last one 2026-06-10, against `recall` at 73 and `decoder` at 61; the suite itself records 12 since 2026-07-03. (2) PREP's stakeholder routing table names people and projects whose working relationships have since ended, so the router's primary case no longer exists. (3) 0 entries in PREP's mandated `Provenance: Verbal, [stakeholder], [date]` capture format across all 13 decisions.md files, against 267 provenance lines carrying a lowercase `verbal` tag, so the capture behaviour happens constantly through the environment's general logging rule and never once through PREP's ritual. (4) The prep-shaped work that actually recurs is interview prep: 6 daily notes in 30 days, four named companies, a document hand-built each time and PREP invoked for none of them. Graded problem evidence partial (real for the interview case, weak for the stakeholder-meeting case) and mechanism evidence none, since nothing tests that a pre-loaded brief changes an outcome. Recommendation is to retire PREP from this suite and park interview-prep-as-a-mode as `status: candidate` rather than build it, which is what step 5's own rule prescribes for real-problem-plus-no-mechanism. Two corrections to a first pass of this entry, both caught by re-running the greps before publication: it claimed 0 verbal-provenance entries outright, from a case-insensitive misread, where the true split is 0 in PREP's format and 267 overall; and it claimed 0 route fixtures cover PREP, where 7 rows cover PREP's trigger phrases (6 HIT, 1 MISS). What is genuinely uncovered is any fixture asserting the MODE is selected, since every row tests suite routing only. The recommendation does not rest on either number.

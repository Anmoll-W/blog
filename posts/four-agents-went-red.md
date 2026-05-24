<!-- source_session: 2026-05-24_vault-consolidation -->

# Four Agents Went RED for Three Weeks. I Cut the Team in Half.

*2026-05-24 · vault, ai-agents, personas, systems · Vault as OS*

For three consecutive weekly evals, the same four agents in my vault came back RED. Not failing in different ways each week. Failing in exactly the same way. The eval agent had documented the gate they were supposed to enforce. The gate was in their memory file. The gate was not being enforced.

When a rule lives in memory and the rule still does not get followed, you do not have a memory problem. You have an architecture problem.

## The Setup

My vault runs on personas — named agents like Maya (content), Alex (engineering), Vera (eval), and so on — each with an identity file, a memory file, and a small routing layer that picks the right one based on what I am about to do. I wrote about the bones of that system in an earlier post on [building an 11-agent AI team](building-an-ai-agent-team.md) and the move from full identity files to lean stubs in [persona-layer-architecture.md](persona-layer-architecture.md).

The system grew. Thirteen personas by May. Four were dormant or near-dormant — the prior week's eval scored Nova, Zara, Sam, and Marco as dormant or dormant-exempt. Routing load on me grew with agent count, because every new task involved a quiet "which agent should this go to" lookup.

That was the cosmetic problem. The real problem was the three RED weeks.

## What the Eval Showed

Vera, the eval agent, runs every Sunday. She scores every other agent on three signals: memory currency, mistake velocity, gate-firing rate. The third signal is the interesting one. A gate-firing rate measures how often the agent actually ran the structured check they were supposed to run before declaring something done.

Four agents were below the threshold for three consecutive weeks. The gate definitions were in their persona files. The gate rules were in their memory. The check was not happening.

The shared-mistakes log captured the diagnosis in one line: *"memory-based template enforcement fails — even for the rule's author."* Five of Vera's own per-agent deep evals from a single week were missing a status block that Vera herself had defined. The rule was in her memory. The rule was not load-bearing on her behavior.

This is the failure mode the Reflexion paper calls *degeneration of thought*. An LLM that reinforces its own pattern across attempts, without external feedback, will keep reinforcing it — even when the pattern is wrong. Memory does not save you. Memory is only as good as the moment at which it is retrieved and the structure that forces it into the output.

## The Three Tracks

I planned the fix as three coupled tracks, run in one sprint.

**Track 1: cut the roster.** Thirteen personas became six: Sage (AI · vault · educator), Alex (engineering · UI/UX), Maya (content · creative), Priya (product · PM all projects), Vera (quality · eval · QA · team health), Rex (business · strategy · money · travel). Seven personas were absorbed into hosts. Their identity files moved to an archive folder. Their HIGH-strength memory entries ported into the host's memory file under a labeled `## Absorbed Domain` section.

Two of those absorptions were tricky. Quinn was a separate QA agent. Merging her into Vera risked losing the test/edge-case lens that ran independently of the decision-quality lens. The fix was to make Vera run as two passes inside one session — a "Quinn pass" first (test and edge cases), then a "Vera pass" (decision quality) — both within the same session. Same persona, two hats, sequenced.

The other one was Lux, the design persona. Lux owned both UI/UX design (component hierarchy, design tokens, dashboard layout) and creative direction (image prompts, quote cards, marketing visuals). Splitting Lux across two hosts only works if the routing matrix knows where to send what. So that became Track 2.

**Track 2: convert skills from sole-owned to shareable.** Of thirty-six skills in the system, only four had any ownership field in their frontmatter — a sole-owner field that locked them to one persona. The other thirty-two had no ownership field at all, which meant routing was implicit and undocumented. Both states are a problem: hard-coded sole-ownership is a bottleneck and a single point of failure; no field at all is worse because there is nothing to read and no signal about which persona should be picking it up.

The new model has three fields per skill: `primary` (the default invoker), `also_invocable_by` (qualified secondary personas), `invocation_rules` (the conditions that route between them). The four legacy-owned skills got their fields rewritten. Everything else got its routing defined in a single matrix file, `skills-routing-matrix.md`, which is now the source of truth for all thirty-six skills × six personas, with primary, secondary, and forbidden cells.

The Lux split lives in the matrix. UI/UX skills route Alex-primary. Creative/visual skills route Maya-primary. Neither host "owns design" as a domain. The matrix routes by skill type. Lux's identity is archived. Lux's rules port into both hosts under labeled sections.

**Track 3: install a brain protocol.** This is the one that actually addresses the RED-week failure.

## The Brain Protocol

Every persona, on every deliverable, now runs a four-step protocol:

**Step 0 — pre-action retrieval.** Before producing any output, grep the persona's own memory file for HIGH-strength entries. Grep the shared-mistakes file for keywords matching the current task. If the deliverable is gated, load the relevant rubric. Total retrieval budget: thirty-five lines, grep-only, cached for the session. No full-file reads. Then name, in conversation, the two most likely failure modes for this task and the load-bearing assumption.

This is the Reflexion fix. The system literature is consistent: anticipating failure modes immediately before producing output reduces the rate of repeating known mistakes. The trick is that the retrieval must be cheap enough to actually happen. Thirty-five lines is cheap.

**Step 1 — ReAct loop.** Think what to produce. Act, produce a draft. Observe, does it match the gate. Reason, what assumption was wrong. Re-act, revise. Loop until the self-observation matches.

**Step 2 — self-critique using a fenced gate block.** Every persona has a named gate block that has to appear in their deliverable. Maya's is `GATE STATUS`. Alex's is `SHIP STATUS`. Vera's is `EVAL STATUS`. Eleven lines for Vera, six to nine for the others. If any line is ❌, the deliverable is not done.

This is the load-bearing change. A rule that lives in memory does not enforce itself. A rule that requires a visible block in the output enforces itself by being structurally required. You cannot ship a deliverable without the block, and you cannot fill in the block without running the check.

**Step 3 — reflect and write.** Three lines back to memory. What I did. What I almost missed or what worked. The rule I am adding, if any. Memory writes land in a `## Pending Validation` section. Vera promotes them weekly. Stale entries decay automatically after sixty days.

A lightweight three-step variant of this protocol runs in sessions where no persona is invoked — when I am working directly with Claude rather than routing to a host. That lite variant lives at the top of the vault's root CLAUDE.md so it loads on every session boot. Grep shared-mistakes for the task keyword. Self-check three lines. Save a feedback rule if one emerges.

## The Audit That Almost Was

After the consolidation ran, I dispatched six audit agents in parallel — one per surviving persona — to find every stale reference in the live vault. The plan's verification phase had specified a routing dry-test with eight signal phrases, all of which resolved correctly. The parallel audit, run in addition, surfaced seven stale references the original sweep had missed.

Most of them were in places the sweep was not designed to look. The CLAUDE.md sweep targeted exactly that filename. It did not touch `index.md` files inside projects, or `master-intelligence.md` (a project-level synthesis doc), or the prose body of the eval rubric file, or the `active-context.md` summary line that gets loaded into every session boot. Across the six audits the findings were: Sage flagged four (an active-context summary line, a thirteen-bullet roster in solopreneur-ai-os, an out-of-date "How to avoid" line in past-mistakes, and one historical append-only log entry that was deliberately left alone because it captured what was true on its date); Maya flagged one (a Lux/Quinn row in the LinkedIn project's master-intelligence table); Priya flagged one (Sam in Ask-the-Cohort's project index, both in the frontmatter summary and the body); Vera flagged one (Quinn in the project rubric's Criterion 2 prose).

Six of those got fixed in the same session. The seventh — an append-only May 12 log entry — stayed intact because it captured what was true on its date. That is the small pattern worth naming: historical logs do not get rewritten, only the lines a future agent will *act on* need to update when the roster changes.

The post-consolidation eval came back six-of-six GREEN.

## What I Will Be Watching for in Week Two

The audits ran with fresh agent instances, no memory pollution from absorbed predecessors. Real personas in real sessions carry memory pollution. Maya, mid-LinkedIn-post, has been writing without owning the image surface area for a long time. Will she correctly invoke her absorbed Lux creative gate without prompting? Or will the muscle memory be "Maya = words, Lux = images" until Vera enforces the gate two or three times?

Week two of the new system is the first natural test of whether the absorbed domains hold under the cognitive load of an actual deliverable. Not whether the routing table is correct. Whether the host persona behaves as if she now owns the absorbed surface.

The thing the eval cannot tell me yet, the thing only published work will tell me, is whether the gate block actually catches the same class of mistake the RED weeks were flagging. The structure says it should. The structure is necessary but not sufficient. I will know in two more Sundays.

## The General Lesson

Three things made the RED weeks resolvable. Reducing the agent count made the system small enough to reason about. Sharing skill ownership made every domain backed up. Installing a structured gate block made the rule load-bearing instead of decorative.

If a rule in memory is not changing behavior, the rule does not exist. Move it into structure — a required block, a fenced template, a step the agent cannot skip without the output being visibly wrong. Memory tells you what to do. Structure makes you do it.

---

## Related

- [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](persona-layer-architecture.md) — the previous redesign of the same layer; this post is the "after" to that one's "before"
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the original roster build; this post is what happened when that roster grew past its useful size
- [The Eval Layer Caught Me Violating My Own Rules](the-eval-layer-caught-me.md) — same eval agent (Vera) catching the same class of memory-vs-behavior failure
- [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](agents-that-dont-learn.md) — the memory layer that this post's Brain Protocol now sits on top of

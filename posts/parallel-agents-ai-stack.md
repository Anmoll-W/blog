<!-- source_session: 2026-03-31_chalo-strategy-ai-stack-audit -->

# Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams

I had a PRD with 6 major feature areas, an 8-day build estimate, and 4 days left before the April 4 deadline. The options were to cut scope, miss the deadline, or find a way to run work in parallel. Cutting scope meant shipping something that was not actually a group travel coordinator. Missing the deadline was not on the table. So I had to figure out what "parallel" actually meant for a single-developer AI-assisted build — and where it would break down if I got the decomposition wrong.

## Mapping the Problem

The first step was decomposing the PRD into workstreams that could genuinely run in parallel. Genuinely parallel means the streams do not need to read each other's output to make progress. Feature areas that share state, share schema, or share core types are not parallel — they are serialized work wearing a parallel costume.

For this project, 6 streams held up:

- Agent 1 (Bot Skeleton): Telegram bot setup, grammY routing, Supabase schema, trip state machine
- Agent 2 (AI Flows): Claude API integration, voice transcription, NLP intent, multilingual handling
- Agent 3 (Core Flows): RSVP flow, budget collection, date alignment
- Agent 4 (Itinerary Engine): AI itinerary generation, constraint checking, Maps deeplinks
- Agent 5 (Expense and Settlement): Bill splitting, UPI deeplinks, sub-group tracking, running balance
- Agent 6 (Scheduled Jobs): Morning briefing, evening wrap, OpenWeatherMap integration, nudge engine

The schema and state machine in Agent 1 were foundational. That one had to land first. Everything else could run after that.

## The Worktree Setup

Each agent gets its own git worktree and a focused CLAUDE.md scoped only to its feature area. The worktree isolation matters: agents cannot interfere with each other because they are in separate filesystem contexts. There is no shared working tree state, no chance of one agent overwriting another agent's in-progress work.

```bash
git worktree add ../chalo-bot-core main
git worktree add ../chalo-ai-flows feature/ai-layer
git worktree add ../chalo-flows feature/user-flows
```

Three terminals open simultaneously. Each runs a focused session. No context switching, no interrupted trains of thought.

## The Constraint That Almost Broke Everything

Parallel agents need shared constraints. Not shared code — shared knowledge about the domain they are operating in.

The constraint that matters for this project: Telegram bots cannot send DMs to users who have not pressed START in the bot's direct message. Every invite into a group flow must use a deep link in the format `t.me/BotName?start=tripId`. A user who has never interacted with the bot directly will silently receive nothing if you try to DM them.

Agent 3 was building the RSVP and preference collection flow. Without knowing this constraint, it would design a flow that sent DMs to group members — a flow that would silently fail for almost everyone in a real group test. The constraint was documented in each agent's CLAUDE.md under a dedicated constraints section before the agents started.

This is the actual design work in a parallel architecture. Not the worktrees. Not the tooling. The hard part is identifying which constraints are cross-cutting and making sure every agent that touches the affected area has them before it starts.

The product decision underneath the parallel setup was this: accept that integration bugs are inevitable on Day 3, budget a full day for them, and treat that day as a required cost rather than a risk to mitigate. The alternative — serializing the work to reduce integration risk — would have blown the deadline. Parallelism was not the safe choice. It was the only choice that fit the constraints, with a deliberate reserve built in for the day it would be most needed.

## The Revised Sprint

Day 1: Agent 1 finishes the bot skeleton and Supabase schema. Agent 2 starts the itinerary engine in parallel, since it only needs the Claude API integration pattern, not the full schema.

Day 2: Agent 3 handles the morning brief and bill split logic. Agent 4 handles UPI settlement and sub-group tracking simultaneously.

Day 3: Real trip test with 3 people. All agents merge to main. Integration bugs surface here — this is expected. Reserve a full day for this.

Days 4 and 5: PRD polish and submission.

## The AI Stack Audit

Running alongside the sprint planning, I audited the full AI tool stack. Two tools got swapped.

Lovable was replaced with Stitch. Lovable generates full-stack output from a prompt: frontend, backend, database. The problem is that the backend is a mystery and the architecture is locked in. Stitch generates design only — UI, HTML, CSS, Figma export. No backend. You own the architecture.

KiloCode was replaced with Claude Code. Claude Code handles agentic project-level workflows better, and the Pro tier is more cost-efficient than raw API usage for heavy daily work.

One addition to the learning system: after every shipped project, drop the PRD and key code into NotebookLM. Generate a 15 to 20 question architecture quiz and a podcast-style audio summary. Then write one vault note: what I mispredicted, and what I would do differently.

The tool pruning rule established during this session: do not add any new tool unless it clearly replaces one of the existing tools in the stack. Addition without subtraction is how a toolchain becomes an overhead.

## What I Learned

Parallel agents require shared domain constraints documented before work starts, not after. The worktree setup is mechanical. Identifying the cross-cutting constraints is the actual design work.

Genuine parallelism requires streams with no shared working-tree dependencies. Streams that share schema or state are serialized work in disguise.

Tool audits have more value when paired with a pruning rule. Adding tools is easy. The constraint that each addition must displace something else forces actual tradeoff thinking.

The decision lesson: when a deadline is fixed and scope cannot move, the PM's job is to find the parallelism — not to work longer hours on a sequential plan. The question is not "how do we go faster?" It is "what work can happen simultaneously, and what must it know before it starts?" Getting that answer wrong does not just slow the build. It produces integration failures that only show up at the worst possible moment — the live test.

---

**[2026-03-31](../README.md#all-posts)** · [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![systems](https://img.shields.io/badge/systems-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Building ChalotripBot →](../series/building-chalotripbot.md)

## Related

- [Live Testing Revealed a Broken Bot](live-testing-revealed-broken-bot.md) — the session that surfaced the failures that made this sprint necessary
- [Building an AI Agent Team](building-an-ai-agent-team.md) — how the parallel agent pattern evolved into a repeatable system

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — the bot rebuilt in 4 days using the parallel agent sprint described here 🔒

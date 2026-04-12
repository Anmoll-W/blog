---
title: "Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams"
date: 2026-03-31
tags: [ai, engineering, product, systems]
series: building-chalotripbot
series_order: 2
source_session: 2026-03-31_chalo-strategy-ai-stack-audit
related: ["2026-03-30-live-testing-revealed-broken-bot.md", "2026-04-10-building-an-ai-agent-team.md"]
---

# Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams

The PRD had 6 major feature areas and an 8-day build estimate. The deadline was April 4. With 4 days left, the math did not work — unless I stopped working sequentially.

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

## Related

- [Live Testing Revealed a Broken Bot](2026-03-30-live-testing-revealed-broken-bot.md) — the session that surfaced the failures that made this sprint necessary
- [Building an AI Agent Team](2026-04-10-building-an-ai-agent-team.md) — how the parallel agent pattern evolved into a repeatable system

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

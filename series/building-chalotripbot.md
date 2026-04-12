# Series: Building ChalotripBot

ChalotripBot is an AI-powered group travel coordinator for Telegram. You add it to a group chat and it handles the parts of trip planning that break down in group settings: aligning dates across schedules, collecting individual preferences privately without exposing them to the group, generating itineraries against real constraints, and tracking shared expenses.

This series documents the build from a broken first version to a working product, including every silent failure that only appeared under real conditions, the sprint strategy that compressed an 8-day estimate into 4 days, and the Telegram platform constraints that forced the core UX to be redesigned before the product could ship.

## Posts in This Series

1. [Live Testing Revealed the Bot Was Fundamentally Broken](../posts/2026-03-30-live-testing-revealed-broken-bot.md)
   The first real multi-user test exposed six critical failures: the primary action silently no-opped, RSVPs only captured the organiser, no one could actually join, voting completed after one response, the waitlist form faked success, and the mobile layout was broken. Every failure was invisible in single-account development testing.

2. [Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams](../posts/2026-03-31-parallel-agents-ai-stack.md)
   After the live test forced a rebuild, the six feature areas were decomposed into genuinely parallel workstreams using git worktrees and focused CLAUDE.md files per agent. The key design work was identifying cross-cutting constraints before the agents started, not after. Also covers an AI tool stack audit that replaced two tools.

## What Is Coming

- The preference collection redesign: how in-group inline keyboards replaced individual DMs and improved completion rates
- The itinerary engine: generating structured multi-day plans against real budget and date constraints using the Claude API
- The expense and settlement layer: bill splitting, UPI deeplinks, and sub-group tracking

---

*[All posts](../README.md)*

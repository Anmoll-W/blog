<!-- source_session: 2026-04-04_in-group-preference-collection -->

# Telegram Bots Cannot DM Users Who Have Not Pressed Start

*2026-04-04 · telegram, silent-bugs · Silent Failures*

I had designed the preference collection flow around DMs because it felt right. Private questions should get private answers. I had tested it myself and it worked. In the first real group test with seven people, one person received the DM. The other six received nothing. No error. No indication anything had gone wrong. The bot continued running as if the flow had succeeded.

The design I had made — DMs per member — was based entirely on testing with my own account, which had always interacted with the bot directly. I had not tested with anyone who had not already pressed START.

## What the Flow Was Supposed to Do

The bot was designed to collect trip preferences from every group member individually. Budget, travel vibe, dietary restrictions. The UX reasoning was sound: you do not want everyone's preferences posted publicly in the group chat. A DM per member felt like the right design.

The implementation sent a direct message to each group member as they were added to the trip. In solo testing, with a single organiser account that had always interacted with the bot, it worked every time.

## The Root Cause

Telegram's API blocks bots from initiating DMs to users who have not previously interacted with the bot directly. "Interacted directly" means the user has opened the bot's DM and pressed START. This is a platform constraint, not a code bug, and it is not surfaced as an error in the API response. The send call returns success. The message is simply never delivered.

This constraint affects the majority of real users. In any group chat where the bot is new, most members will not have pressed START. The flow was designed around an assumption that was false for almost everyone it would ever encounter.

## The Redesign

The fix was not a patch on the existing flow. The entire preference collection pattern had to change.

All interactive collection now happens directly in the group chat via inline keyboards. The bot posts a card: "Trip to Goa — tap to set your preferences." Each member taps the card in the group. Telegram delivers a private per-user popup natively, so responses are still not visible to other group members. The group sees a shared counter that updates as each person submits: "3 of 7 budgets submitted."

The final flow:

1. Bot posts RSVP card in the group
2. Each member taps in the group chat
3. Telegram delivers a private inline popup to that member only
4. Member selects their response
5. Group counter updates

No DMs required. No deep links required for preference collection. Anyone in the group can respond immediately.

## How the Implementation Ran

The redesign was scoped as 12 focused tasks: update the RSVP handler, update the budget handler, update the vibe handler, update the dietary handler, add group counter logic, update the state machine, and so on. Each task was small enough to complete without full codebase context.

All 12 tasks ran in a single session using a subagent approach. Each subagent received a focused brief covering only the task it needed to complete. No subagent needed to understand the full system. The session went from redesign decision to merged and deployed in one sitting.

## The Broader Pattern

Platform constraints that are invisible during solo development become critical at the first real multi-user test. I had documented the Telegram DM constraint during planning. Documenting a constraint is different from designing for it.

The original flow was designed by someone who was always the organiser and had always pressed START. That person was me. The constraint was abstract knowledge until a real group test made it concrete.

The principle that follows: test with users who are not you, at the first possible opportunity where the platform constraint could manifest. For any bot flow that sends messages to users, that test needs at minimum two accounts — one that has pressed START and one that has not. The second account is the one that reveals the failure.

## What I Learned

Telegram DM blocking is silent. The API returns success. There is no error to catch. The only signal is that users never respond — which looks identical to users choosing not to respond.

In-group inline keyboards with per-user popups are a cleaner pattern for group preference collection than individual DMs. The group counter creates social accountability that actually improves completion rates.

Document platform constraints in a testable form, not just a note form. "Bots cannot DM users who have not pressed START" is a note. "Before shipping any flow that sends DMs to group members, test with one account that has never pressed START" is a test.

**A design tested only by the person who built it is not tested.** I had written down the DM constraint during planning. I had read about it. I still shipped a flow that failed for six out of seven users because every test I ran used my account — the one account guaranteed to work. The decision going forward: any flow that requires prior user action to succeed needs to be tested by someone who has not taken that action. That test cannot be done by the developer.

---

**[2026-04-04](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![telegram](https://img.shields.io/badge/telegram-0088cc?style=flat-square&logoColor=white)](../README.md#all-posts) [![silent-failures](https://img.shields.io/badge/silent--failures-d73a4a?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Silent Failures →](../series/silent-failures.md)

## Related

- [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](telegram-privacy-mode.md) — another platform constraint that only surfaces in real group contexts
- [Claude Appends Text After JSON](claude-appends-text-after-json.md) — a different class of silent failure, same session
- [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](three-cascading-bugs.md) — three more failures that were invisible during development
- [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) — what happens when a fix addresses symptoms rather than cause
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a rendering-layer failure with no visible error

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — the bot where this Telegram constraint broke the preference collection flow 🔒

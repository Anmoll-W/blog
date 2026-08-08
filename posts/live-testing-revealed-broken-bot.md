<!-- source_session: 2026-03-30_bug-fixes-and-deploy -->

# Live Testing Revealed the Bot Was Fundamentally Broken

*2026-03-30 · engineering, testing · Building ChalotripBot*

I had shipped a group travel coordinator to real people and told them to try it. I had tested it myself, from my own account, as the organiser, and it had worked. What I had not done was test what happened when a second person tried to join. Or RSVP. Or do anything that required them to not be me.

The first real test with actual users exposed that the core flows were not working at all. "Plan a trip" did nothing. Only one RSVP had ever been recorded. No one could join the trip. This is ChalotripBot, an AI-powered group travel coordinator for Telegram, and the fundamental mechanics of group coordination were broken from the start. The decision to ship before multi-user testing was the decision that made all of this visible at the worst possible moment — in front of real users instead of in a staging environment.

## What Failed and Why

**The primary action silently no-opped.** The "plan a trip" command had an `if (!trip)` guard that blocked trip creation if any existing trip record was found for the group in the database. Since there was already a record from earlier testing, every new attempt silently did nothing. No error, no response, no feedback to the user. The fix was to check for an existing trip and respond with status and options rather than halting.

**RSVPs only captured the organiser.** Non-invite-link members had no row in the members table. The update call ran against zero rows and returned silently. Everyone who joined the group through a normal Telegram link was invisible to the RSVP system. The fix was an upsert that inserts a row if one does not exist before updating it.

**No one could actually join.** The invite link flow required pressing START in a direct message with the bot before the user could participate. Real users joined the Telegram group but never sent a DM to the bot. There was no prompt, no guidance, and no handler for the `new_chat_members` event. The fix was adding a handler that sends the invite link prompt automatically when someone joins the group.

**Voting completed after one response.** The vote completion check compared against `confirmedMembers.length`, which was 1 because only the organiser was in the database due to the RSVP bug above. With one confirmed member, a single vote satisfied the threshold. The fix involved the RSVP upsert bringing real numbers in and adding a `Math.max(2, ...)` floor so voting cannot complete with a single participant.

**The waitlist form was faking success.** The submit handler ran a `setTimeout` and displayed a success state without calling the API. No one's email address was being saved. The form looked functional and felt functional. No one knew it was not working.

**The mobile layout was broken.** The waitlist form had three elements in a single flex row: name input, handle input, and button. On phones, the button was completely off-screen.

## Why These Bugs Survived Until Production

Every one of these bugs was invisible during development because development used controlled inputs from a single account: the organiser's.

The RSVP upsert bug only appears when a second person tries to RSVP. The invite link bug only appears when someone joins the group without pressing START first. The vote threshold bug only appears when more than one person is in the member database. The waitlist form bug only appears when you trace the submit handler instead of watching the UI.

Testing the happy path from the organiser's perspective exercises roughly half the system. The join flow, the RSVP flow for latecomers, and any state that requires multiple participants are all invisible from a single-account test.

The product decision embedded here was implicit rather than conscious: ship when the organiser flow works, gather feedback on the rest. That is a reasonable decision in some contexts. In a group coordination product, where the entire value proposition depends on the second and third user having a working experience, it was the wrong one. The non-organiser flow was not a stretch goal. It was the product.

## Two Build Failures From a Parallel Merge

After fixing the bot logic, pushing to deploy triggered two cascading build failures that had nothing to do with the bot fixes. Both were merge artifacts from a previous merge where two development threads had touched the same file. One thread had renamed a variable. The other had not. The merge completed without conflict but produced code that referenced the old variable name in one place and the new name everywhere else.

The build error pointed to an undefined variable. The fix was straightforward once the cause was clear. But the diagnosis took longer than it should have because the error appeared immediately after deploying fixes and looked like something the fixes had introduced.

Parallel development on the same file, even when the merge appears clean, can produce build failures that only surface at compile or deploy time. The merge resolution itself is not the right verification point. A successful build after the merge is.

## What I Learned

Live testing with real users who are not the builder reveals failure paths that single-account testing structurally cannot. The join flow is the most important flow to test and also the hardest to test alone.

Silent no-ops are corrosive to user trust in a way that errors are not. An error gives the user something to report. A silent no-op gives them nothing, and they leave assuming the product does not work, because it does not.

Tracing a form submit handler to the database write, rather than watching the UI, is the only reliable way to verify that a form is working. A convincing success state proves nothing about what happened in the backend.

The decision lesson is harder to swallow: for any product where the core value requires more than one user, "working" is not defined until the second user has a complete experience. Shipping before that bar is met is not an early release — it is shipping an untested product. The checklist question I would now add before any group-feature release: can someone other than me, joining the system cold, complete the primary flow without any intervention from me?

---

**[2026-03-30](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![testing](https://img.shields.io/badge/testing-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![telegram](https://img.shields.io/badge/telegram-0088cc?style=flat-square&logoColor=white)](../README.md#all-posts) [![debugging](https://img.shields.io/badge/debugging-586069?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Building ChalotripBot](../series/building-chalotripbot.md)

## Related

- [Three Cascading Bugs](three-cascading-bugs.md) — silent failures at the infrastructure level, same session context
- [Telegram Privacy Mode](telegram-privacy-mode.md) — another class of silent failure from the same live testing session, this time at the Telegram delivery layer
- [Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams](parallel-agents-ai-stack.md) — the sprint strategy this session made necessary
- [Telegram Bots Cannot DM Users Who Have Not Pressed Start](telegram-bots-cant-dm.md) — the follow-on Telegram platform constraint that reshaped the preference collection flow
- [Layover: Guess Where You Woke Up](layover-guess-where-you-woke-up.md): the next chapter under the same Certified Lost brand, a solo-shipped daily game and the launch-day fix that closed its own version of a live-testing surprise

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**Project:** [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — the bot whose first live test is documented in this post 🔒

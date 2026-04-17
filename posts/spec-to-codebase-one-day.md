<!-- source_session: 2026-04-10_design-and-build -->

# From Problem Space to Working Codebase in One Day

*2026-04-10 · engineering, product, ai*

The PM Learning Companion was a buildathon product — two days to go from nothing to something reviewable. Day 1 was research and problem framing. Day 2 had a specific definition of done: get 16 screens navigable, every user flow walkable, the product reviewable by a non-technical teammate. The risk was the same one every Day 2 carries — spending the day building backend infrastructure and ending with zero screens anyone can see, or spending it building screens against assumptions that the data layer would later contradict. The decision that made this day work was refusing to let either of those failure modes happen. That refusal has a name: mock-first. But it also has a design counterpart: persona separation. Both were product decisions before they were technical ones.

By midnight, 16 routes were locally functional. No database, no real API keys, no deployment. But every screen existed, every user flow navigated, and the product felt like something you could actually use. This post is about how it worked.

## The Persona Separation That Made It Possible

The day split into two distinct modes: designer in the morning, engineer in the afternoon. These were not the same person wearing different hats. They were treated as separate agents with separate mandates, separate context, and separate outputs. The designer does not think about implementation constraints. The engineer does not reconsider design decisions.

This separation matters more than it sounds. When one person holds both roles simultaneously, they make premature tradeoffs. The designer self-censors because they are also the one who has to build it. The engineer rewrites screens mid-build because the design feels wrong in context. Separating the roles, even when the same human is running both, removes that interference.

The morning produced a full design brief: color tokens, typography scale, component library specs, emotional specs for all 13 screens, animation specs, and edge case handling. The afternoon consumed that brief as an input and produced a working codebase.

## What the Design Brief Locked Down

The design brief answered one question for every screen: what does this screen need the user to feel, and what does it need them to do next?

A few decisions from that session that shaped the entire product:

The archetype reveal screen was designed like Spotify Wrapped. Full-screen, high-contrast, identity-forward. The reasoning was that archetype assignment is a product moment that most tools waste. If you tell someone they are a Sprinter with four weeks to their interview, that should feel like a diagnosis, not a form confirmation. The screen needed weight.

The quiz was designed to be inescapable. It appears inline after an article and cannot be closed or skipped. Two questions maximum. The constraint is intentional: the product thesis is that comprehension verification is the missing loop, so the product cannot allow users to route around it. If you close the quiz, you close the learning. The design enforces the thesis.

The Scanner path got a completely different layout: magazine grid, no progress bar, no lock mechanic. Every other archetype path has structure and scarcity (one article unlocked, one blurred below it). The Scanner path looks like a publication. The differentiation is visual before it is functional, because the Scanner will not tolerate being treated like a student.

The onboarding used no forms. Single question per screen, visual tiles only. The reasoning was that the product is for working PMs who are already skeptical of onboarding flows. Every extra field is a signal that the product does not know how to prioritize.

## The Stack Decision

Next.js 16, React 19, TypeScript, Tailwind version 4, Supabase, Anthropic SDK, Framer Motion, Resend, Vercel. Nothing exotic. The constraint was buildathon speed and the assumption that the reviewer would be a PM, not an engineer. The stack needed to be deployable, not clever.

The Anthropic SDK was in the stack from the start because two routes required language model evaluation: summary assessment (did the user actually understand the article?) and synthesis assessment (for the Scanner archetype, open-ended comprehension). Both of those routes were specced on Day 2, implemented on Day 2, and mocked on Day 2. The API calls were written. The prompt templates were designed. The keys were not yet wired.

That sequencing was intentional.

## The Mock-First Philosophy

The rule for the build session: get every route functional with mock data before wiring any real data source. Do not block route completion on data availability.

The feed used a hardcoded article object. The archetype derivation used rule-based logic (if onboarding answer equals interview, archetype is Sprinter) rather than the Claude API call that was already specced. The Day 7 streak check was hardcoded to false. The Supabase fetch was wired in the code but pointed at a table that did not exist yet.

This approach produces something that looks wrong to an engineer and feels right to a product person. The code is full of TODO markers and hardcoded values. But the product is navigable. A non-technical teammate can click through the entire user journey and give real feedback. That feedback is more valuable at Day 2 than any data layer correctness.

The product decision: prioritize navigability over correctness, and name that decision explicitly before the session starts — not as a shortcut, but as the goal. If the goal had been left implicit, the pull toward "doing it right" would have reasserted itself mid-session. The Supabase schema is important. It is also the wrong thing to build on Day 2 when the question being answered is "does this product make sense to a user?"

The honest near-miss: the instinct at the start of the afternoon was to set up the Supabase schema first. It felt irresponsible to hardcode data when the schema was already designed. The rule saved the day. If that instinct had been followed, the session would have ended with correct fetch logic and no Sprinter archetype reveal screen. Nobody would have been able to see whether the "Spotify Wrapped" moment worked. That is a Day 2 failure, regardless of how clean the database layer is.

The alternative is building the database schema first. You spend a day on Supabase, get the schema right, wire the fetch functions, write the auth flows. By end of Day 2 you have a solid backend and zero screens. Nobody can see what you are building. Nobody can challenge your assumptions about what the Sprinter archetype reveal screen should feel like. You have built infrastructure for a product you have not yet shown anyone.

## One Pattern Worth Naming

Before calling the Claude API to evaluate a user's article summary, the code checks whether the summary is mostly copied from the article. Length heuristic plus sentence overlap. If it looks like a paste, the route rejects it without making an API call.

This is not primarily a cost optimization, though it does save tokens. It is about closing the most obvious gaming path before the product is in anyone's hands. If you build a comprehension gate and the first thing a skeptical user does is paste the article back, the gate fails. The check runs before the model sees anything.

The pattern generalizes: for any AI evaluation in a product, ask what the simplest bypass looks like and close it in code before the model sees the input. The model is not the right place to catch verbatim pastes. Deterministic checks are cheaper and more reliable.

## What Was Not Done at End of Day 2

No environment file. Supabase and Anthropic keys not configured. No Supabase schema. No deployment. No re-engagement email templates in Resend.

These were not failures. They were the correct scope for Day 2. The gap between a locally functional product and a deployed one is real work, but it is a different category of work from the gap between zero screens and 16 screens. Day 2 closed the harder gap.

## What I Learned

The design session and the build session are most valuable when they happen on the same day in sequence. If a week passes between design and build, the builder has lost the emotional context for what each screen needs to feel like. The design brief captures the decisions but not the reasoning that was alive in the room when they were made. Same-day sequencing keeps that reasoning accessible.

Mock data first is not a shortcut. It is a deliberate prioritization of navigability over correctness. A product you can navigate produces better feedback earlier. Correct data in a product no one has seen yet is expensive inventory.

The separation of designer and engineer personas prevents the most common Day 2 failure mode: designs that get simplified during build for reasons that have nothing to do with user needs and everything to do with the builder's energy level.

The decision lesson: "mock-first" has to be a named decision made before the session, not a principle invoked mid-session when the database setup is taking longer than expected. The difference is whether it is a strategy or a rationalization. Making it explicit at the start — "Day 2 goal is navigability, not correctness, and that is correct" — is what prevents the mid-session drift back toward infrastructure work. The same principle applies to any product build under a time constraint: the definition of done has to be set before the session, not negotiated during it.

---

**[2026-04-10](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![buildathon](https://img.shields.io/badge/buildathon-586069?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [Research Before Building: How I Map a Problem Space from Scratch](pm-problem-space-research.md) — the Day 1 research session that preceded this build
- [Running Parallel Agents on a Real AI Stack](parallel-agents-ai-stack.md) — the infrastructure approach behind running designer and engineer as separate agents

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

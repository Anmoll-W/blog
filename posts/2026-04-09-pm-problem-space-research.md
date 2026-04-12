---
title: "Research Before Building: How I Map a Problem Space from Scratch"
date: 2026-04-09
tags: [product, ai, pm, research]
series: ""
series_order: 0
source_session: 2026-04-09_problem-space-research
related: ["2026-04-10-spec-to-codebase-one-day.md"]
---

# Research Before Building: How I Map a Problem Space from Scratch

The worst research habit I have seen in product work is inheriting a problem statement and treating it as settled. Someone writes a document, it circulates, it earns a few comments, and then the team starts building against it. The document was never validated. The archetypes in it were never stress-tested. The problem framing was never challenged. It just accumulated gravity by existing.

This post is about what I did instead, and what it revealed.

## The Starting Point

This was Day 1 of the PM Learning Companion buildathon (Rethink Systems Cohort 7, April 9-14, 2026). The team had a secondary research document with a problem statement and three user archetypes: Climber, Specialist, Scanner. Reasonable starting point. The document was well-written. The problem framing was credible.

I set it aside and rebuilt the analysis from scratch using live web research across eight dimensions: market size for PM learning tools, the PM skills gap between what employers report and what PMs believe about themselves, content consumption habits, newsletter readership patterns, the forgetting curve and knowledge degradation timelines, promotion failure patterns from hiring managers, the rate at which AI is shifting the required PM skill set, and interview preparation behavior.

The goal was not to disprove the original document. The goal was to arrive at the problem statement independently, so that any agreement would be grounded rather than inherited.

## What the Research Produced

Three documents came out of this session: a problem analysis with a root cause, a six-phase journey map, a severity matrix, a competitor gap analysis, and a locked problem statement. A target group document with eight user groups, six archetypes, and an intervention map. A customer journey map for the Sprinter archetype across seven stages.

The problem statement that survived:

"PMs fail to develop through content consumption because no product sequences what to read, verifies comprehension, or signals skill growth. The consumption loop is complete. The learning loop has never been built."

That is the sentence everything else anchors to.

## The Archetype Expansion

The original document had three archetypes. After mapping the full PM population, the model expanded to six: Mapper (breaking into PM from another field), Climber (12-month promotion track), Sprinter (active interview preparation, 4-6 week window), Specialist (domain deepening in an area like growth or data), Coach (teaching others, often a senior IC or manager), and Scanner (VP/CPO maintaining epistemic hygiene across a wide surface area).

The expansion matters less than what the expansion revealed. The original three archetypes treated career level as the primary variable. Mapper was early career. Climber was mid-level. Scanner was senior. The model was a ladder.

That framing is wrong. Archetypes are goal crossed with urgency, not career levels. A VP who is interviewing is a Sprinter. A new PM who wants a promotion in 12 months is a Climber. Career level sets the default, but the actual onboarding question determines the archetype for that session. The design implication is significant: you cannot assign an archetype from a LinkedIn profile. You have to ask.

## The Surprise About Who Pays

My initial assumption was that the Climber would anchor the paid tier. Promotion anxiety is real, the 12-month window creates sustained urgency, and mid-level PMs are a large and well-defined market.

Research pushed back on this. Interview prep is a proven paid category with demonstrated willingness to pay, shorter decision cycles, and clear success criteria. Exponent, Interview Kickstart, Lenny's interview prep content: these businesses exist because the Sprinter pays before the Climber does. The Sprinter has a deadline. The Climber has a goal. Deadlines convert better.

The decision was to anchor the paid tier on the Sprinter, not because Climbers are unimportant, but because the Sprinter's urgency structures the entire product experience in a way that benefits every other archetype. Build for the person with the shortest time horizon and the highest stakes. The other archetypes find their way in through a product that already knows how to move people forward.

## The VP Use Case Nobody Builds For

The Scanner archetype surfaced a positioning angle no competitor has taken.

The core problem for a VP or CPO is not that they need to learn new things. They read constantly. They absorb signals from many directions. The problem is that nobody stress-tests their models. A VP can hold a belief about, say, PLG motion or discovery process quality, that was formed three years ago and has never been challenged since. The information they consume confirms more than it interrogates.

Comprehension gates are irrelevant to this archetype. You do not quiz a CPO on whether they understood an article. What you build for the Scanner is an environment where their synthesis gets challenged by a system that has read the same material and asks better follow-up questions than any direct report would.

No competitor in the space addresses this. The Scanner is a strong differentiated positioning angle precisely because it requires a product model that is completely different from the standard "read, quiz, repeat" loop.

## The Method That Made This Possible

Running the research across eight independent dimensions in parallel meant that convergences emerged organically. The forgetting curve data aligned with the content consumption habits data: PMs read a lot and retain almost none of it, not because they are lazy but because the format is wrong. No sequencing, no spacing, no retrieval practice. The problem is not motivation. The problem is structure.

That convergence would not have appeared if I had accepted the original framing and gone straight to solution space. The original document described the problem correctly but did not explain why the problem persisted. Eight independent research threads gave the why.

## What I Learned

The archetypes that get cut matter as much as the archetypes that survive. When you trim a user population to three personas, you are making a claim about who the product is not for. That claim should be deliberate, not a side effect of stopping research early.

Secondary research shapes thinking before it is validated. The original document was not wrong, but it was accepted at face value by the team. Rebuilding it from scratch found the same problem statement through a different path, which strengthened confidence in the statement, and found the archetype gaps that the original had missed.

The paid-tier assumption is almost always wrong on the first pass. Who you think will pay is usually the person who talks loudest about the problem. Who actually pays is usually the person with the hardest deadline.

## Related

- [From Problem Space to Working Codebase in One Day](2026-04-10-spec-to-codebase-one-day.md) — what happened on Day 2, when the research turned into a running application

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

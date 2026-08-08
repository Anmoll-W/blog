<!-- source_session: 2026-04-08_github-profile-rebuild -->

# Building the GitHub Profile README: Design Decisions and the One Constraint That Shaped Everything

*2026-04-08 · github, portfolio · GitHub is Your Portfolio*

The strategic call had been made: archive the React portfolio site, make GitHub the primary presence. Now the profile README had to actually carry that weight. The risk was doing what most developers do by default — open the file, start writing, and end up with something that reads like a formatted CV. I had just archived a polished portfolio site for this. A CV-shaped README would make that tradeoff pointless.

The first thing I did was not write markdown. It was run a structured design session to decide what the profile should feel like before a single line of content got written. That sequence matters. Writing before deciding on voice and structure produces something that reads like a first draft forever.

## The Three Formats Considered

Three directions were on the table: a minimal editorial style (single column, sparse text, clean), a data dashboard (GitHub stats widgets, contribution graphs, activity heatmaps), and a narrative story format where the profile reads like a PM presenting work in a review.

The data dashboard was rejected immediately. GitHub stats widgets measure commit frequency, not impact. A streak counter tells a reader nothing about whether the work mattered. Every developer profile with these widgets looks identical, and the ones doing the most interesting work often look the worst because they are managing products rather than pushing code every day. Rewarding quantity over substance is a design anti-pattern, not just an aesthetic choice.

The minimal editorial style has real appeal. Clean, hard to get wrong, does not overreach. But it leaves the reader doing all the interpretive work. A PM portfolio needs to show thinking, not just list outputs.

Narrative story won. The profile should read like a PM presenting their work to a senior stakeholder: context first, then what was built, then why it mattered.

This was not a neutral aesthetic preference. Rejecting the data dashboard was a product decision about what signal to send. Every PM who has had to argue against vanity metrics in a product review has been in this same position: the number is right there, it is easy to show, and it measures the wrong thing. Choosing not to display commit streaks on a PM portfolio is the same call. The metric you choose to display shapes how a reader interprets everything else on the page.

## The Specific Decisions Made

The opening block uses a terminal or neofetch-inspired YAML structure as a hook. Not a gimmick. A signal that the person writing this profile is comfortable with code environments, and a fast way to surface the facts (current role, scale, stack) before the narrative begins. The YAML block is a hook. The content below it is the actual portfolio.

The projects section is public repos only, framed as case studies. Not a list of repository names with auto-generated descriptions. Each project gets a one-line problem statement, what was built, and the result. The experience section is separate, ordered chronologically, and kept factual. These two sections do different jobs and should not be merged.

Testimonials appear from two people with specific credentials. Not generic praise. The quotes are selected because they reinforce the positioning: a PM who builds. One quote comes from a senior stakeholder at a company known for engineering culture. The other comes from a senior PM who managed me directly. The specificity is the point.

The stack section is ordered deliberately: AI tools first (Claude Code, Cursor), then analytics, then development tools, then product tools. The ordering signals where the actual work happens. Listing Figma after Claude Code is not an accident. It reflects how I actually spend time building.

The profile has no portfolio site link, no PDF resume link, no "contact me" section with six icons. One LinkedIn link at the bottom. The profile is complete on its own. Every external link you add is a reader you are sending away.

## The Constraint That Made Everything Clearer

GitHub profile READMEs render inside a fixed-width container with no custom CSS available. The only design primitives are standard markdown: headers, bold, tables, code blocks, and shields.io badges. That is the entire toolkit.

This constraint removes a category of decisions that usually consume time and produce weak outcomes. You cannot compensate for thin content with a strong visual hierarchy. You cannot rescue a shallow project description with a nice layout. The profile lives or dies on the quality of the writing and the logic of the structure.

I find this clarifying rather than limiting. The best constraint in a design process is one that removes false choices without removing real ones. A fixed-width markdown container does exactly that. The decisions that remain, voice, structure, ordering, framing, are the decisions that actually determine whether the profile works.

The writing rules I imposed on myself (no em dashes, no shorthand symbols, no abbreviations) were not arbitrary. Markdown on GitHub renders in a monospace or near-monospace reading environment depending on the block type. Typography discipline affects how text reads at a glance, even when the typography tools available are limited.

## What the Design Session Produced

A profile that reads as a product artifact rather than a CV update. The difference is not cosmetic. A CV lists what someone has done. A product artifact makes an argument: here is the work, here is the thinking behind it, here is the through-line that connects everything.

That argument can be made in markdown inside a fixed-width container. The constraint does not prevent it. In some ways, the constraint enforces it.

## What I Learned

Running a visual design session before writing produces better structure. Deciding on voice, format, and what to exclude before touching content saves revision cycles later.

Rejecting GitHub stats widgets is not a preference, it is a product decision. The metric you choose to display shapes how a reader interprets everything else on the page. Commit frequency is the wrong metric for a PM portfolio.

The constraint of standard markdown is a feature. It forces every design decision to be a content decision.

The decision lesson: the format you choose is the argument you are making before anyone reads a word. A data dashboard argues "I commit frequently." A narrative profile argues "I think clearly and I know what my work means." Before choosing any format for any artifact — a deck, a README, a spec — the question worth asking first is: what claim does this format make by itself? Getting that question wrong is how you end up with a polished artifact that signals the opposite of what you intended.

---

**[2026-04-08](../README.md#all-posts)** · [![github](https://img.shields.io/badge/github-24292f?style=flat-square&logoColor=white)](../README.md#all-posts) [![portfolio](https://img.shields.io/badge/portfolio-24292f?style=flat-square&logoColor=white)](../README.md#all-posts) [![personal-branding](https://img.shields.io/badge/personal--branding-586069?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [GitHub is Your Portfolio](../series/github-is-your-portfolio.md)

## Related

- [GitHub Is Your Portfolio](github-as-your-portfolio.md) — the strategic decision that preceded this build session
- [Why I Rebuilt My GitHub Profile as a Wikipedia Article](wikipedia-style-readme.md) — the next iteration, two days later, and why the format changed again

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**Project:** [Anmoll Wadhwa](https://github.com/Anmoll-W) — the profile README built in this post

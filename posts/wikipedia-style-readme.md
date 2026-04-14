---
date: 2026-04-10
tags: [github, portfolio, personal-branding]
series: github-is-your-portfolio
---
<!-- source_session: 2026-04-10_wikipedia-style-readme -->

# Why I Rebuilt My GitHub Profile as a Wikipedia Article

Two days after shipping the narrative portfolio README, I rebuilt the profile again. Not because the first version was wrong, but because I had been looking at the wrong model. The narrative format worked as a standalone document. It did not work as the entry point to a network.

The gap became visible when I started thinking about the pinned repos. Each of them tells a story. But none of them connected back to the profile, none of them connected to each other, and there was no shared vocabulary across them. The profile was an article with no links out and no links in. Wikipedia is the inverse of that. Every article links outward and receives links in. The structure of the network is inseparable from the quality of any single page.

That observation drove the rebuild.

## What Changed Visually

The most visible change is the right-floated infobox, built as an HTML table using the `align="right"` attribute since GitHub markdown does not support CSS positioning. The infobox contains the facts: headshot, current role, the scale range of products shipped (1 million to 90 million MAU), what I am known for, education, and two links (LinkedIn and resume). It sits in the top right of the page. A reader scanning quickly can get the full factual summary from the infobox without reading a word of the narrative.

This is intentional. The infobox and the narrative serve two different reading patterns. Someone arriving with five seconds and a specific question (who is this, what scale have they worked at, where did they study) can get that from the infobox without reading prose. Someone arriving with genuine curiosity reads the narrative. Both readers are served without forcing either to adapt to the other.

The headshot is hosted directly in the repo at `assets/headshot.png` and referenced via a raw.githubusercontent.com URL. Not a CDN link, not an image hosting service. CDN links from third-party services expire when accounts change or services shut down. A raw GitHub URL is permanent as long as the repo exists. That is the correct tradeoff.

The hatnote is a small italic disambiguation line above the article title, borrowed directly from Wikipedia conventions: "This article is about Anmoll Wadhwa the product manager. For the Telegram travel bot, see ChalotripBot." It signals immediately that there are other articles in this namespace. The disambiguation note is doing architectural work, not comedic work.

The table of contents is a collapsible block using `details` and `summary` HTML tags, which GitHub markdown renders correctly. Nine sections, several with subsections. The section names follow Wikipedia conventions: About, Career, Work That Shipped, Tools and Stack, Thinking in Public, Education, Certifications, See Also, External Links.

The grey shields.io category badges at the bottom classify the article: Product Management, AI, Consumer Scale, Creator Economy, Indian PMs. These are not skill tags. They are classification categories, the same way Wikipedia articles have category footers. They will appear on every repo README in the network.

## The Hub Architecture

The Wikipedia metaphor is not only visual. It describes an information architecture.

A single well-written profile README is a good article. It does not feel like Wikipedia. Wikipedia feels like Wikipedia because of the density of the link graph. Every article links to related articles. Every subject has a category. Clicking into any node takes you somewhere relevant. The network is navigable.

The current state of my GitHub presence has one article (the profile) and several unconnected nodes (the repos). The gap analysis from the rebuild session was direct about what is missing and what each gap requires.

Repo READMEs need to be case studies, not auto-generated descriptions. Each pinned repo should have a README structured as: Problem, Build, Results, Learned. That makes each repo an article with substance, not a stub.

Each repo README needs a back-link to the profile. Wikipedia articles do not exist in isolation. A reader who arrives at a repo directly (from a search, from a share, from another person's reference) should be able to navigate back to the main article with one click.

Related repos should reference each other in a See Also section. This is what creates the sense that the repos exist in a coherent namespace rather than as independent projects.

The same grey category badges should appear on every repo README. Consistent classification across the network makes the categories meaningful. A badge that appears once is decoration. A badge that appears on every node in the network is a navigation system.

## The Shortest Path to a Real Wikipedia Feel

The ChalotripBot repo is the obvious next step. The hatnote already references it by name, which creates a broken link in the current state. Writing the ChalotripBot case study README would give the network its second article. Two interconnected articles, each linking to the other, each linking back to the profile. That is the minimum viable version of the architecture.

Once that pattern exists, every future repo gets built the same way. The pattern is the asset, not the individual README. A consistent format means each new project inherits the navigation structure automatically rather than requiring a separate design decision.

## What I Learned

The shift from narrative format to Wikipedia format was not an aesthetic upgrade. It was a structural one. The first profile was a document. The second one is a node in a network that does not fully exist yet but now has a defined architecture.

The infobox solves a real problem: two readers arrive at a profile with different intentions and different amounts of time. Serving both without asking either to adapt requires separating the fact layer from the narrative layer. The infobox is the fact layer.

Hosting assets inside the repo and referencing them via raw.githubusercontent.com is the correct choice for any image that needs to be permanently available. The short-term convenience of an external CDN link is not worth the long-term fragility.

## Related

- [Building the GitHub Profile README](github-profile-rebuild.md) — the design decisions in the session that preceded this one
- [GitHub Is Your Portfolio](github-as-your-portfolio.md) — the strategic framing that started this series

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [Anmoll Wadhwa](https://github.com/Anmoll-W) — the GitHub profile rebuilt as a Wikipedia article in this post

<!-- source_session: 2026-04-08_codebase-audit-bug-fixes -->

# How to Audit a Production Codebase Against Its Own Support Data

*2026-04-08 · engineering, debugging*

Before this audit session, the plan was to start shipping bug fixes. The plugin had unresolved support volume and the pressure to ship something — anything — was real. The problem was that without knowing which areas of the codebase were causing the most pain, any fix would be a guess. Shipping a guess when you have 1,165 support interactions already telling you exactly where users are hurting is a product decision, not just a technical one. This audit was not optional cleanup. It was the gate before any fix could be trusted.

This post walks through a method for using support ticket categories as a map into an unfamiliar codebase, what patterns make PHP bugs invisible in production, and how to tell a real bug from a false positive.

## Start With the Category Distribution, Not the Tickets

The first step is not reading individual tickets. Individual tickets are anecdotes. You want the distribution.

For a WordPress SEO plugin with 1,165 support interactions across chat, email, and chatbot channels, the breakdown looked like this: AI Features accounted for 27.1% of chat tickets. Technical-Linking accounted for 26.9% of chatbot conversations. These were the two largest non-billing categories. That is where the audit started.

The insight is simple. If 27% of your support volume points at a specific feature area, there is a statistically high probability that something is structurally wrong in that code. You read the code in those areas first. Everything else waits.

The product decision here is worth naming: the audit determined the sequencing of every fix that followed. Without it, the instinct would have been to start with the most recently reported bug, or the easiest one to fix, or whichever one the support team mentioned last. All three of those are the wrong sequencing criteria. The distribution is the right one. Deciding to trust the data over intuition meant the highest-impact fixes went out first rather than the most convenient ones.

This is the opposite of how most engineers approach a codebase. The default is to start at the entry point, follow the architecture, understand the whole system. That is fine for onboarding. It is not fine for finding bugs that are actively hurting users. For that, you need to start where the pain is.

## What Makes PHP Bugs Invisible in Production

After reading the AI Features and Technical-Linking code against that support data, several bugs emerged. The pattern across most of them is the same: they fail silently.

The SQL escaping bug in `Keyword.php` is a clean example. Keywords containing apostrophes (common in English, universal in French and Spanish) were interpolated directly into SQL without escaping. When `$wpdb->get_results()` receives malformed SQL, it returns an empty array. No exception. No log entry. No visible error. The auto-linker runs, finds nothing, inserts nothing, and the user sees a feature that appears to work but produces no output. A 15% estimate of affected users is conservative.

The double-wrapped JSON in `Maintenance.php` follows the same pattern. `wp_send_json([$plan])` wraps the response in an extra array layer. The JavaScript client expects an object and receives an array of arrays. Every property access returns `undefined`. The maintenance process appears to start and then hangs. From the user's perspective: the button worked, something happened, and then nothing. No error state. No feedback. Just silence.

The stale lock TTL in `AI.php` is the same class of problem. A 24-hour lock on the embedding index means that if an out-of-memory error kills a run mid-batch, the next cron run picks up the same stale lock and potentially OOMs again. Users see "AI calculations in progress" indefinitely. The lock is doing its job. The TTL is wrong. And because the error path is silent, the user cannot distinguish "working slowly" from "permanently stuck."

The pattern across these bugs: the failure state looks like normal operation. No exception is raised. No log is written. The system keeps running. The user receives a degraded or broken experience with no indication of why.

## Cross-Referencing Code With User Reports

Once you have identified a candidate bug in the code, the next step is finding a user report that matches it. This is the verification step. If the behavior you found in the code maps precisely to a ticket category, you have high confidence it is real. If you cannot find a matching report, you should slow down.

The honest version of this step: before doing it, the assumption was that the support tickets were already well understood. Someone had categorized them, the categories had names, and the top categories were known. It felt like the data was being used. It was not. Knowing that "AI features" was the top category is not the same as knowing that keywords with apostrophes produce broken SQL and silent empty results. The gap between category label and root cause is where the audit actually lives.

For the SQL escaping issue: the ticket category "auto-linking not working" matches exactly. A keyword with an apostrophe produces broken SQL, which produces an empty result, which produces no links. The user reports that auto-linking is not working. The code explains the report.

For the maintenance hang: the ticket category "AI features stuck" and "stuck at 1%" match. The double-wrapped response means every property access fails, so the progress indicator never advances. The user reports that the feature is stuck. The code explains the report.

The license grace period bug (10 consecutive server failures triggering auto-deactivation) maps to tickets about unexpected deactivation with no warning. The code explains the report.

This cross-reference is the difference between a bug you found and a bug that is causing production pain. Both matter, but they have different urgency levels.

## Distinguishing Real Bugs From False Positives

Two candidates in this audit turned out not to be bugs.

`Dashboard.php:1100` initially looked like a potential null crash. The method `getLinks()` is called on what could be a null object. Reading further into `Model/Post.php` at lines 175-183, `getLinks()` always returns an object with empty string fallbacks. The crash path does not exist.

`Suggestion.php:193` had an unconditional `break` inside a while loop, which looks like dead code or a logic error. Reading the surrounding context: each AJAX call is designed to process exactly one batch, increment a counter, and return the counter to the client for the next call. The while loop exists as a time-limit guard. The break is intentional. The design is correct.

The discipline here is: flag it, read the full context, check if there is a corresponding user report, and rule it out explicitly before moving on. Do not add it to the fix list until you have done that work. False positives waste engineering time and can introduce regressions when you "fix" code that was working correctly.

## The 80/20 of Codebase Audits

The goal of an audit driven by support data is not completeness. A complete audit of a large plugin codebase would take weeks and would surface hundreds of minor issues. That is not useful.

The goal is finding the 20% of bugs responsible for 80% of the support volume. Support categories give you a ranked list. You read the top categories first, find the structural bugs in those areas, verify them against user reports, and ship fixes. Then you move to the next category.

Five bugs fixed in a single session, directly traceable to the top two support categories. Three more identified and queued for a test environment. Two false positives ruled out with confidence.

That is what using support data as a map looks like in practice.

## What I Learned

The most dangerous production bugs are the ones that produce no errors. Silent failure means users cannot report the actual problem, only the symptom. Searching for "no error thrown, unexpected empty result" is a productive audit strategy for any PHP codebase using WordPress database abstractions.

Support category distribution is a better audit entry point than code architecture documentation. The categories reflect where users are actually experiencing pain, not where the engineers think the complexity is.

Ruling out false positives is as important as finding real bugs. Fixing code that is working correctly based on a surface-level read is worse than leaving it alone.

The decision lesson: knowing a category name is not the same as understanding a category. Before this audit, the support data felt like it was informing product decisions. It was not — it was being read at the label level, not the root-cause level. The audit forced the question: "what in the code would produce exactly this symptom?" That question is different from "what do users complain about?" and it is the one that actually drives sequencing. Any time a roadmap is being prioritized using support data, the right check is whether the underlying mechanism is understood, not just the category label.

---

**[2026-04-08](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![debugging](https://img.shields.io/badge/debugging-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![wordpress](https://img.shields.io/badge/wordpress-586069?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](support-tickets-root-causes.md) — the follow-up session where these findings were extended into a full fix batch
- [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) — one of the fixes in this audit (SQL escaping) had an error that was caught in review the next day
- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md) — the audit surfaced the category taxonomy that later shaped the AI digest prompt; this post covers what the dashboard became after nobody logged in
- [We Were Paying for a Support Tool. Gmail Already Did the Job.](gmail-support-system-without-paid-tools.md) — the audit method is the upstream of the support system: understanding ticket categories is what makes routing filters worth building
- [What I Learned Auditing an AI Agent Repository Built by Another Product Manager](auditing-another-pms-agent-repo.md) — the same read-the-evidence-first discipline applied to another person's agent code instead of an unfamiliar production codebase

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Projects:** [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) 🔒 · [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) 🔒

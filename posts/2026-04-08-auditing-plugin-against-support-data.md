---
title: "How to Audit a Production Codebase Against Its Own Support Data"
date: 2026-04-08
tags: [engineering, product, debugging, wordpress]
series: ""
series_order: 0
source_session: 2026-04-08_codebase-audit-bug-fixes
related: ["2026-04-08-support-tickets-root-causes.md", "2026-04-09-when-the-ai-fix-is-wrong.md"]
---

# How to Audit a Production Codebase Against Its Own Support Data

Most codebases are too large to audit from scratch. The question is never "where might bugs exist" but "where do bugs exist that are causing pain right now." Support data answers that question directly, and almost nobody uses it that way.

This post walks through a method for using support ticket categories as a map into an unfamiliar codebase, what patterns make PHP bugs invisible in production, and how to tell a real bug from a false positive.

## Start With the Category Distribution, Not the Tickets

The first step is not reading individual tickets. Individual tickets are anecdotes. You want the distribution.

For a WordPress SEO plugin with 1,165 support interactions across chat, email, and chatbot channels, the breakdown looked like this: AI Features accounted for 27.1% of chat tickets. Technical-Linking accounted for 26.9% of chatbot conversations. These were the two largest non-billing categories. That is where the audit started.

The insight is simple. If 27% of your support volume points at a specific feature area, there is a statistically high probability that something is structurally wrong in that code. You read the code in those areas first. Everything else waits.

This is the opposite of how most engineers approach a codebase. The default is to start at the entry point, follow the architecture, understand the whole system. That is fine for onboarding. It is not fine for finding bugs that are actively hurting users. For that, you need to start where the pain is.

## What Makes PHP Bugs Invisible in Production

After reading the AI Features and Technical-Linking code against that support data, several bugs emerged. The pattern across most of them is the same: they fail silently.

The SQL escaping bug in `Keyword.php` is a clean example. Keywords containing apostrophes (common in English, universal in French and Spanish) were interpolated directly into SQL without escaping. When `$wpdb->get_results()` receives malformed SQL, it returns an empty array. No exception. No log entry. No visible error. The auto-linker runs, finds nothing, inserts nothing, and the user sees a feature that appears to work but produces no output. A 15% estimate of affected users is conservative.

The double-wrapped JSON in `Maintenance.php` follows the same pattern. `wp_send_json([$plan])` wraps the response in an extra array layer. The JavaScript client expects an object and receives an array of arrays. Every property access returns `undefined`. The maintenance process appears to start and then hangs. From the user's perspective: the button worked, something happened, and then nothing. No error state. No feedback. Just silence.

The stale lock TTL in `AI.php` is the same class of problem. A 24-hour lock on the embedding index means that if an out-of-memory error kills a run mid-batch, the next cron run picks up the same stale lock and potentially OOMs again. Users see "AI calculations in progress" indefinitely. The lock is doing its job. The TTL is wrong. And because the error path is silent, the user cannot distinguish "working slowly" from "permanently stuck."

The pattern across these bugs: the failure state looks like normal operation. No exception is raised. No log is written. The system keeps running. The user receives a degraded or broken experience with no indication of why.

## Cross-Referencing Code With User Reports

Once you have identified a candidate bug in the code, the next step is finding a user report that matches it. This is the verification step. If the behavior you found in the code maps precisely to a ticket category, you have high confidence it is real. If you cannot find a matching report, you should slow down.

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

## Related

- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](2026-04-08-support-tickets-root-causes.md) — the follow-up session where these findings were extended into a full fix batch
- [When the AI Fix Is Wrong](2026-04-09-when-the-ai-fix-is-wrong.md) — one of the fixes in this audit (SQL escaping) had an error that was caught in review the next day

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

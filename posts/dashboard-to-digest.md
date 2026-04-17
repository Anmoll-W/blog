<!-- source_session: 2026-04-13_digest-pipeline-smoke-test -->

# Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead

## What I Built (and Why It Was Wrong)

Twelve days ago I built a support dashboard. I had just shipped it. I thought it was useful. When I checked the access logs to see who was using it, the answer was nobody. The dashboard had three users. Two of them were me on different machines. The third had not logged in since the day I sent the link.

The thing I had not asked before building it: what job does this tool need to do, and does a dashboard actually do that job?

The dashboard answered: "What does the last month of support data look like if you navigate to a URL and interact with filters?" That is a legitimate question. It was not the question anyone on the team was asking. The question that actually needed answering was: "What do I need to act on this morning, before my first call?" Those are different products. One lives in a browser. The other lives in your inbox. I had built the browser one — charts, category breakdowns, response time percentiles, ticket volumes by day, filters and drill-downs — without verifying that anyone wanted to visit a URL to get this information.

The decision I had deferred was whether to make support data available or make support data useful. I assumed they were the same problem. They are not. Access to data means building a place where data can be viewed. Making data useful means delivering the right signal to the right person at the right moment, without requiring any action from them. A dashboard requires the user to decide to open it, navigate to it, and then figure out what they are looking at. A digest removes all three steps.

## What I Changed

I deleted the Supabase database entirely and replaced the whole system with a single cron job.

The cron runs at 01:30 UTC — 7 AM IST — every morning. It:

1. Fetches the last 24 hours of support tickets from HelpScout via the v2 API
2. Passes them to Claude Haiku with a structured prompt: summarise top issue categories, flag refund signals, note anything unusual in volume or tone
3. Sends a plain HTML email and a Slack message with the digest: ticket count, open tickets, resolution rate, SLA breaches (open tickets older than 24 hours), and the AI-generated paragraph

No database. No UI. No navigation. The summary arrives in your inbox before your first call. You either read it or you do not — but you do not have to remember to go somewhere.

Total infrastructure cost: roughly one dollar a month in API calls.

## What I Learned

**The job to be done is not "view dashboard."** It is "know what to act on this morning." Designing around the wrong job produces the wrong product. The technical lesson: before instrumenting any data pipeline, write out the specific moment of use — who is reading this, when, and what decision does it need to enable? If that moment requires the user to navigate somewhere, the delivery mechanism is probably wrong.

**Storage costs are a product signal.** The database was growing because every ticket was being persisted on the assumption that it might be needed someday. It never was. If infrastructure is growing while usage is flat, that is not a capacity problem — it is a design problem. The data was being stored for a use case that did not exist.

**Deletion is a product decision.** Removing the database was the highest-leverage thing shipped that week. Not a new feature, not an optimisation — a subtraction. The instinct in product work is almost always to add. The decision-making principle it exposed: before building any internal tool, ask whether the right move is to build at all, or whether the problem disappears if you change the delivery mechanism. The dashboard was not a bad implementation. It was the wrong answer to a question nobody had asked out loud.

## The HelpScout API Problems

Building the replacement exposed four failure modes in the HelpScout v2 API that were not documented clearly anywhere.

**Vercel outbound IPs are blocked from the OAuth token endpoint.** When the cron runs on Vercel, the outbound request comes from an AWS IP range. HelpScout's `/v2/oauth2/token` endpoint returns `invalid_client` for requests from those IPs — silently, with no indication that the block is IP-based rather than credential-based. The same credentials work from a local terminal or a GitHub Actions runner. Fix: pre-generate a token locally, store it as a Vercel environment variable, refresh it daily via a GitHub Action that runs before the cron fires.

**`createdAtStart` and `createdAtEnd` cause HTTP 500.** The natural API for fetching tickets in a time window is a date range. HelpScout's v2 accepts these parameters in documentation but returns a 500 with no useful error body when they are used. The only working parameter is `modifiedSince`. Filtering to the actual window has to happen client-side after fetching.

**`embed=threads` causes HTTP 500 on some accounts.** Embedding thread data in the conversations response is documented and looks like it should work. It does not — at least not on this account. The endpoint returns 500 regardless of other parameters. Switched SLA breach detection to an age-based proxy: any open ticket older than 24 hours counts as a breach.

**ISO 8601 dates must not include milliseconds.** The `modifiedSince` parameter rejects `2026-04-12T00:00:00.000Z` but accepts `2026-04-12T00:00:00Z`. The failure mode is a 400 error with a message that does not point to date format. Fix: strip `.000Z` from any ISO timestamp before it goes into a query parameter.

The pattern across all four: the error describes what failed, not why, and gives no direction on what to try instead. The only way through was testing each parameter variant in isolation.

## The Token Refresh Architecture

The pre-generated token approach introduces an expiry dependency: HelpScout tokens have a 48-hour TTL. If the token is not refreshed, the cron silently sends no digest.

The fix is a GitHub Action at 01:00 UTC — 30 minutes before the cron:

1. Requests a fresh token from HelpScout (GitHub Actions IPs are not blocked)
2. Updates `HELPSCOUT_ACCESS_TOKEN` in Vercel using the CLI
3. Triggers a Vercel production redeployment to pick up the new value

The system is fully autonomous. No manual token management, no expiry monitoring, no intervention needed.

---

**[2026-04-13](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![helpscout](https://img.shields.io/badge/helpscout-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![vercel](https://img.shields.io/badge/vercel-24292f?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [Building an Internal Support Dashboard: From Broken Scaffold to Live Data](building-internal-dashboard.md) — the session where the dashboard was originally built; this post is the sequel showing what replaced it
- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](support-tickets-root-causes.md) — what the support data actually contained, which informed the digest's category groupings
- [How to Audit a Production Codebase Against Its Own Support Data](auditing-plugin-against-support-data.md) — using support ticket analysis to prioritise engineering work
- [When the AI Fix Is Wrong: What Senior Review Catches](when-the-ai-fix-is-wrong.md) — on verifying that a fix is addressing the real cause, not just the visible symptom
- [We Were Paying for a Support Tool. Gmail Already Did the Job.](gmail-support-system-without-paid-tools.md) — the next step after the digest: replacing the paid shared inbox entirely with a Gmail-native triage system built from filters, delegation, and templates

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — the repository where this digest pipeline lives 🔒

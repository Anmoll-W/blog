---
title: "Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead"
date: 2026-04-13
tags: [engineering, product, ai, helpscout, vercel]
series: ""
series_order: 0
source_session: 2026-04-13_digest-pipeline-smoke-test
related: ["building-internal-dashboard.md", "support-tickets-root-causes.md", "auditing-plugin-against-support-data.md", "when-the-ai-fix-is-wrong.md"]
---

# Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead

Three months ago I built a support dashboard. Charts, category breakdowns, response time percentiles, ticket volumes by day. It pulled data from the support inbox, stored it in Supabase, and surfaced it in a Next.js interface.

Last week I deleted the database entirely.

Not because the dashboard broke. Because when I checked the access logs, nobody had logged into it in weeks — including me.

This post is about what I built instead, the three product lessons that came out of it, and the HelpScout API failures that made the replacement harder than it should have been.

## The Wrong Question

The dashboard was answering the wrong question.

The question it answered: "What does the last month of support data look like if you navigate to a URL and interact with filters?"

The question that actually needed answering: "What do I need to act on this morning, before my first call?"

Those are different products. One lives in a browser. The other lives in your inbox.

Nobody on the team needed to log in and explore data. They needed a summary, delivered, every day, without any action on their part. A dashboard optimises for exploration. What was needed was delivery.

I killed the dashboard and built a digest instead.

## What the Digest Does

A single cron job runs at 01:30 UTC — 7 AM IST. It:

1. Fetches the last 24 hours of support tickets from HelpScout via the v2 API
2. Passes the raw ticket list to Claude Haiku with a structured prompt asking for insight — top issue categories, refund signals, anything unusual in volume or tone
3. Sends a plain HTML email and a Slack message with the digest: ticket count, open count, resolution rate, SLA breaches (open tickets older than 24 hours), and the AI paragraph

Total infrastructure: one Vercel cron route, one GitHub Action for token refresh, no database. Cost: roughly one dollar a month in API calls.

The digest goes out every morning whether anyone thinks about it or not. That is what the dashboard was never going to do.

## Three Product Lessons

**The job to be done is not "view dashboard."** It is "know what to act on this morning." These produce entirely different designs. One requires a user to navigate somewhere and ask a question. The other requires no decision from the user at all. When designing internal tools, the question worth asking first is: what is the actual job? Not "how do we surface the data" but "what action does this data need to enable, and how close to that action does the tool need to be?"

**Storage costs are a product signal.** The Supabase database was growing because every ticket was being persisted on the assumption that it might be needed later. It never was. If infrastructure is growing while usage is flat, that is not a capacity problem — it is a design problem. The data was being stored for a use case that did not exist. Deleting it was not a loss; it was a correction.

**Deletion is a product decision.** Removing the database was the highest-leverage thing shipped that week. Not a new feature, not an optimisation — a subtraction. The instinct in product work is almost always to add. But the right decision is sometimes to find the thing that should not exist and remove it cleanly. Deletion requires clarity about what the product is actually for.

## The HelpScout API Problems

The replacement looked straightforward on paper. In practice, the HelpScout v2 API had four separate failure modes that were not documented anywhere obvious.

**Vercel outbound IPs are blocked from the OAuth token endpoint.** When the cron runs on Vercel, the outbound request comes from an AWS IP range. HelpScout's `/v2/oauth2/token` endpoint returns `invalid_client` for requests from those IPs — silently, with no indication that the block is IP-based rather than credential-based. The same credentials work from a local terminal or a GitHub Actions runner. The fix: pre-generate a token locally, store it as a Vercel environment variable, and build a GitHub Action that refreshes it daily before the cron fires.

**`createdAtStart` and `createdAtEnd` cause HTTP 500.** The natural API to use for fetching tickets in a time window is date range parameters. HelpScout's v2 API accepts them in the documentation but returns a 500 with no useful error body when they are used. The only working parameter is `modifiedSince`. Filtering to the specific time window has to happen client-side after fetching.

**`embed=threads` causes HTTP 500 on this account.** Embedding thread data in the conversations response is documented and looks like it should work. It does not. The endpoint returns 500 regardless of other parameters. Thread data has to be fetched per-conversation in a separate request — or skipped entirely. We skipped it and switched SLA breach detection to an age-based proxy: any open ticket older than 24 hours counts as a breach.

**ISO 8601 dates must not include milliseconds.** The `modifiedSince` parameter rejects `2026-04-12T00:00:00.000Z` (with milliseconds) but accepts `2026-04-12T00:00:00Z` (without). The failure mode is a 400 error with a message that does not mention date format. The fix is a one-line helper that strips `.000Z` from any ISO timestamp before it goes into a query parameter.

Each of these failures was silent in a specific way. The IP block looked like a credential error. The `createdAtStart` failure looked like a server error with no cause. The `embed=threads` failure looked identical. The millisecond issue looked like a malformed request but gave no guidance on which part was wrong. The pattern across all four: the error response described what failed, not why, and not what to do instead.

## The Token Refresh Architecture

The pre-generated token approach introduces a new operational dependency: the token has a 48-hour TTL and expires if not refreshed.

The solution is a GitHub Action that runs at 01:00 UTC — 30 minutes before the cron fires at 01:30 UTC:

1. Requests a new access token from HelpScout using the client credentials flow (GitHub Actions IPs are not blocked)
2. Updates the `HELPSCOUT_ACCESS_TOKEN` environment variable in Vercel using the CLI
3. Triggers a Vercel production redeployment to pick up the new value

The cron always has a fresh token. The system is fully autonomous — no manual token management, no expiry alerts, no intervention needed.

## What Was Kept

The dashboard code is still in the codebase, dormant. The API routes exist. The Supabase client library is uninstalled and the database is gone, but the query logic is there. Reviving it would require one environment variable and one database provisioning step.

This was a deliberate choice. Deleting code that might be useful in a different form is different from deleting infrastructure that has a carrying cost. The database had a cost. The code has none.

## Related

- [Building an Internal Support Dashboard: From Broken Scaffold to Live Data](building-internal-dashboard.md) — the session where the dashboard was originally built; this post is the sequel
- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](support-tickets-root-causes.md) — what the support data actually contained, which informed the digest's category groupings
- [How to Audit a Production Codebase Against Its Own Support Data](auditing-plugin-against-support-data.md) — using support ticket analysis to prioritise engineering work
- [When the AI Fix Is Wrong: What Senior Review Catches](when-the-ai-fix-is-wrong.md) — on verifying that an apparent fix is actually addressing the real cause

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — the repository where this digest pipeline lives 🔒

<!-- source_session: 2026-04-22_ga4-digest-shipped -->

# Google Analytics Was Fine. Opening It Was Not.

*2026-04-22 · engineering, product, ai, analytics*

## Same Pattern, Different Data

Nine days ago I deleted a support dashboard and replaced it with a daily cron-generated digest. Nobody was opening the dashboard. The digest started arriving in the inbox at 07:00 IST and the problem disappeared: you either read it before your first call or you did not, but you did not have to remember to go somewhere.

This week I ran the same play on Google Analytics.

The analytics dashboard was not broken. The data was clean, the property was correctly configured, the instrumentation had years of history behind it. The thing that was broken was the habit. No one on my team opened `analyticsdata.google.com` on a normal Tuesday. By the time someone opened it, it was usually because something had already gone wrong and they needed to go back and check what yesterday's data had looked like. The question being asked was not "let me explore the charts". It was "tell me what changed yesterday and whether I should act on it."

That is not a dashboard. That is a daily email.

## What I Built

One cron endpoint, one email, one piece of AI in the middle. The architecture maps exactly to the support digest that shipped last week, with different data:

```
Vercel Cron (09:00 UTC daily, chosen so the GA4 property's LA-timezone "yesterday" is fully closed before we query it)
  → GA4 Data API (service-account auth, Analytics Viewer on the property)
  → Aggregator: yesterday snapshot, funnel vs baseline, channels, devices, top 10 landing pages
  → Claude Haiku: five-field insight block, written as the product manager
  → nodemailer → Gmail SMTP → recipient list
```

Six sections in the email, in order: yesterday snapshot (sessions, users, transactions, CVR, revenue with week-over-week arrows); funnel vs baseline (homepage to pricing to checkout to confirmation, each step flagged against a stored baseline); top five channels with conversion rate; device split (mobile CVR is highlighted in red if it drops below one percent); top ten landing pages with bounce and engagement; a flags section that is empty and green when everything is within baseline, and a coloured chip list when it is not. Below all of that, a five-line insight block written by Claude Haiku in the voice of the product manager.

No database. No UI. Same delivery stack as the support digest: shared SMTP credentials, shared recipient list, shared cron authentication secret. The only new external surface is the GA4 Data API.

## The Prompt Is the Product

The interesting part is not the cron. Crons are cheap. The interesting part is that a generic analytics summary would be noise. "Sessions were 491 yesterday, conversions were 6, revenue was $594" is not an insight. It is a recital.

What makes the email useful is that Haiku reads the numbers as a specific PM with a specific set of priorities. The prompt begins:

> You are Priya, Senior PM for the product. Your 2026 priorities in order: renewal rate up ten percentage points, grow organic 3x, revenue per user. Locked CRO priorities: P1 pricing page, P2 first-visit bounce, P3 blog conversion path. Mobile CVR has historically been 0.3 percent. Treat any mobile regression as a P0 flag.

Then five structured fields: productSignal, topAction, trend, pageToWatch, channelStory. Each field has a specific job and a concrete example. The topAction field is not allowed to say "monitor" or "review": it has to name a thing to audit, a test to run, or a person to ask.

With that framing, yesterday's email read: *"Pricing to Checkout dropped from 38.2 percent baseline to 31.1 percent, a minus 18.6 percent regression. Install Clarity on /pricing/ today so we can see where the 55 percent who do not progress to checkout drop out."* Not a summary. A next action.

The same data with a generic prompt would have told me the sessions count and stopped there.

## Three Gotchas That Cost Me an Hour Each

**The GA4 API rejects `dateRange` as a dimension even though the response includes it.** The SDK allows you to request multiple date ranges in one query. When you do, the response has a pseudo-dimension called `date_range_0`, `date_range_1`, etc., but you do not list it in the `dimensions` array. Listing it throws `INVALID_ARGUMENT: Field dateRange is not a dimension`. The fix is to drop the line. The response still distinguishes rows by the pseudo-dimension; you just do not declare it.

**The purchase page path was wrong.** The funnel proxy was built against four paths: homepage, pricing, checkout, and thank-you. The site uses `/checkout/purchase-confirmation/` for the thank-you page, not `/thank-you/`. The first live run showed Checkout-to-Purchase at zero percent against a 21.2 percent baseline. Haiku immediately flagged it as the top regression, which was useful pressure testing of the prompt, but also a reminder that any URL-based funnel in an email is fragile to site changes. A twenty-line GA4 query confirmed the actual path; the fix was a two-character diff.

**Two Vercel projects share similar names.** The deployment target was `linkwhisper-support-dash.vercel.app`. I spent ten minutes hitting `support-dash-ruby.vercel.app`, a different project from an earlier iteration that still carries the same route shape but has the wrong environment variables. The new route responded with 401 because the `CRON_SECRET` the old project had was no longer current. Vercel's deployment URL shows up in the project list; `vercel project ls` was the fastest way to confirm which project owned the alias before wasting more curl calls.

None of these would have been caught by tests. The tests passed with 53 green. The gotchas only surfaced on live data from the actual property.

## What the Pattern Is

Both digests, support and analytics, follow the same shape: a data source that is nominally "available", a habit that never forms to check it, and a product manager's question that the dashboard is not designed to answer. The answer in both cases is the same pipeline: fetch, aggregate, narrate as a specific person with a specific set of priorities, email.

The delivery mechanism is the product. The dashboard is a data interface. The digest is a decision interface. Switching one for the other does not require new data: it requires noticing that the question people are actually asking is not the question the dashboard was built to answer.

Next up in this stack is probably a weekly competitor digest: what moved in the category, who shipped what, what the content rankings did. Same pattern. Different data. Different PM voice.

---

## Related

- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md): the original move from dashboard to digest, with the pattern spelled out for support data.
- [We Were Paying for a Support Tool. Gmail Already Did the Job.](gmail-support-system-without-paid-tools.md): the layer below this digest; Gmail labels as the data source before anything else was built.
- [The Same Pattern Twice: Building an Internal Support Dashboard From Broken Scaffold to Live Data](building-internal-dashboard.md): the earlier dashboard that eventually got deleted, useful as the "before" that motivated the pattern change.
- [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](claude-appends-text-after-json.md): the kind of Claude API gotcha that pairs with the GA4 prompt defensiveness described above.
- [The Metric Did Not Improve. The Denominator Changed.](the-denominator-changed.md): a separate GA4 case, two reports with the same metric label and two different denominators, both technically correct and mutually incomparable.
- [Analytics Said Zero. The Order System Said Otherwise.](analytics-said-zero.md): a GA4 report showing four straight months of zero conversions, and the order system that proved the zero was an attribution gap, not an absence of sales.

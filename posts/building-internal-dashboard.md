<!-- source_session: 2026-04-01_supportdash-initial-setup -->

# Building an Internal Support Dashboard: From Broken Scaffold to Live Data

*2026-04-01 · engineering, supabase, nextjs*

The goal for this session was to build a support dashboard that would surface ticket trends in realtime — a tool that could tell, at a glance, which feature areas were generating the most pain and whether a fix had actually moved the number. The business case was straightforward: stop guessing about support volume and start reading it. The risk was equally straightforward: ship a broken data layer and every insight the dashboard produces is wrong. Before writing a single feature, the foundation needed to be verified.

The codebase already existed. An AI tool had scaffolded it, the folder structure looked reasonable, and the first instinct was to start building features. The second instinct, which arrived about ten minutes later, was to check what database driver the scaffold had actually used.

It had used SQLite.

## The Problem With Inheriting a Scaffold

AI tools that scaffold full-stack apps make database choices for you, often without surfacing them. This one had generated a SQLite wrapper appropriate for local development or a prototype. The production database was Postgres on Supabase. Before any feature work could happen, the entire data layer needed to be torn out and rebuilt.

This is not a complaint about the scaffolding tool. It is a reminder that inheriting a scaffold means inheriting its assumptions. The first step in any session on an inherited codebase is to verify those assumptions against the actual deployment environment.

The migration involved:

- Replacing the SQLite wrapper with Drizzle ORM and a native postgres driver
- Migrating every table definition: `sqliteTable` to `pgTable`, integer timestamps to proper `timestamp()` types, integer booleans to native `boolean()`, `real()` to `numeric()`
- Setting up two connection strings: `DATABASE_URL` pointing to the pooler on port 6543 for the app, and `DATABASE_URL_DIRECT` pointing to port 5432 for migrations only

The two-connection-string pattern is a Supabase requirement, not a preference. Supabase Transaction mode does not support prepared statements. Any connection through the pooler needs `prepare: false` set explicitly. Miss that and the app fails on the first parameterized query with an error that does not obviously point to the connection configuration.

## The Surprise: No Sync Script Needed

Before the session started, the plan included writing a sync layer to pull support tickets from the external ticketing platform into the dashboard's database. This felt like a significant piece of work — polling logic, deduplication, incremental sync, error handling.

Halfway through the data layer migration, while reading the existing Supabase schema, it became clear that the ticketing platform was already writing to Supabase in realtime. The integration had been built previously and was running. The database already had the data.

This changed the scope of the session entirely. Instead of building a sync layer, the work was to understand the existing table structure and build a query layer on top of it. Two to three hours of planned work evaporated. The right response was to update the plan immediately rather than keep building toward a problem that did not exist.

The product decision: drop the sync layer entirely and treat the existing integration as the source of truth. This is a decision worth naming because the alternative — building the sync layer anyway for "completeness" or to match the original plan — is a real failure mode. Plans are wrong. When the evidence says a piece of planned work is unnecessary, the correct move is to update the plan, not defend it. The dashboard was going to be used for support triage, not as an exercise in building sync infrastructure. The scope that served that goal was the right scope.

## The Silent Bugs

Getting to a running dashboard against real data involved four bugs that shared a common property: none of them produced an obvious error message.

The first was an environment variable loading failure. The `.env.local` file was written with heredoc indentation, which put leading whitespace before every variable name. Next.js silently ignores environment variables whose names begin with whitespace. The app loaded, reported no errors, and had no configuration values.

The second was a type coercion issue with the `numeric` type in Postgres. Drizzle returns `numeric` columns as strings, not numbers. Any arithmetic on those fields produces `NaN`. The value looks like a number, the operation looks correct, and the output is silently wrong. The fix is a `parseFloat(String(val))` call before any calculation on those fields.

The third was a server-side rendering mismatch. Calling `toLocaleTimeString()` inside a server-rendered React component causes a hydration error because the server and the client format times differently based on locale. The fix is to move any time formatting into a `useEffect`, keeping it client-side only.

The fourth was a connection string encoding issue. The database password contained an `@` character. In a connection string, `@` is the delimiter between credentials and host. A literal `@` in the password breaks the parser without an error message that points to the connection string. The fix is to URL-encode `@` as `%40`.

What these four bugs have in common: they all fail silently. The app loads. No exception is thrown. The failure mode is missing data, wrong data, or a subtle rendering inconsistency. The only way to catch them is to verify outputs explicitly rather than assuming a lack of errors means correct behavior.

## End State and What Was Missing

The session ended with the dashboard running locally against 763 rows of seeded data in Supabase. Seven API routes were live. The data layer was solid.

What was discovered at the very end of the session: the dashboard had no authentication. It was publicly accessible. This had not been part of the original scaffold and had not been part of the session plan. The next step was clear — add authentication before mapping the real production table structure.

That near-miss is the relatable moment from this session. The dashboard was running. It was showing real data. The session felt done. The only reason authentication was caught was that the end-of-session checklist included security review. Without that check, a dashboard with 763 rows of live support data would have been left publicly accessible while the next session moved on to feature work. The plan had no auth step because the scaffold had not included one, and the assumption was that the scaffold had handled the basics. It had not.

The lesson from the session order: migrations first, then verify data, then map real structure, then auth. In practice, auth nearly got skipped entirely because it was not in the original plan and was only discovered by accident. It belongs in the plan from the beginning.

## What I Learned

Inheriting a scaffold means auditing its assumptions before writing a line of feature code. Database choice, connection configuration, and environment variable handling are worth checking in the first ten minutes.

Silent failures require explicit output verification. An app that loads without errors is not the same as an app that is working correctly. Checking actual rendered values and actual database query results is not optional in early setup sessions.

Discovering that a problem does not exist is a good outcome, not a wasted plan. The sync layer was dropped cleanly once the evidence showed it was unnecessary. Updating the plan and moving on was faster than building something that was not needed.

The decision lesson: a plan built before a session starts does not know what the session will find. The sync layer was on the plan because the assumptions going in were wrong — the integration was assumed to not exist. When those assumptions were disproved, the plan should have been updated immediately, not preserved. Any session plan for infrastructure work should include an explicit verification step at the start: "does this problem already exist?" Security requirements like authentication should be in the plan by default, not discovered by accident at the end. If the checklist had not caught it, it would have shipped without it.

---

**[2026-04-01](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![supabase](https://img.shields.io/badge/supabase-3ecf8e?style=flat-square&logoColor=white)](../README.md#all-posts) [![nextjs](https://img.shields.io/badge/nextjs-000000?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai](https://img.shields.io/badge/ai-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) — what happens when you trust the AI's diagnosis without verifying it against the actual system state
- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md) — what happened three months later: nobody used the dashboard, so the database was deleted and replaced with a 7 AM digest cron

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — the dashboard built in this post, now live 🔒

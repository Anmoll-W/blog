---
date: 2026-04-09
tags: [code-review, ai-coding, wordpress, debugging, silent-bugs, php]
series: silent-failures
---
<!-- source_session: 2026-04-09_pr6167-cto-review-corrections -->

# When the AI Fix is Wrong: What Senior Review Catches That Pattern Matching Misses

> The AI found real bugs. It also introduced three new ones. All three had the same root cause.

## Context

In April 2026 I submitted a pull request to the LinkWhisper WordPress plugin codebase. The PR contained 9 bug fixes across multiple files, all generated with AI assistance by tracing 1,165 support tickets back to root causes in the PHP code.

Most of the fixes were correct. The CTO reviewed the PR, made corrections via a separate commit directly to the branch, and provided detailed feedback on what had been wrong.

Three fixes were reverted. One was corrected. The pattern behind all four errors was the same: the AI pattern-matched on the surface appearance of the code without tracing data lifecycle, reading domain semantics, or checking the math.

This post documents each case in detail. If you are using AI tools for code review or bug fixing, this is what you need to watch for.

## Error 1: SQL Escaping Without Tracing Data State

The AI added `esc_sql()` around a keyword variable before a database query:

```php
$escaped_keyword = esc_sql($keyword->keyword);
```

This looks correct. You should always escape data before SQL queries. The problem is that `$keyword->keyword` was not raw data.

At line 1596 of the same file, there was an explicit `str_replace("\'", "'", $keyword->keyword)`. That line exists because WordPress magic quotes had already been applied to the data. The string entering the escaping function looked like `don\'t`, not `don't`.

Applying `esc_sql` on top of magic-quoted data produces `don\\'t`. The database search now looks for a literal backslash, finds nothing, returns no results.

The correct fix is to remove the magic-quote slashes first:

```php
esc_sql(wp_unslash($keyword->keyword))
```

The AI did not trace where the data came from or what transformations had already been applied to it. It saw "database query, no escaping" and added escaping. The fix was technically a fix for the surface problem and a new bug underneath.

**Before adding any escaping to a codebase you did not write, search for `wp_magic_quotes`, `stripslashes`, `sanitize_text_field`, and `wp_unslash` in the data path first.**

## Error 2: Lock Time-to-Live Semantics Misread

The AI shortened an embedding lock time-to-live from `DAY_IN_SECONDS` to `HOUR_IN_SECONDS * 4`:

```php
// Before (correct)
set_transient($lock_key, true, DAY_IN_SECONDS);

// After (wrong)
set_transient($lock_key, true, HOUR_IN_SECONDS * 4);
```

The AI's reasoning was that a 24-hour lock seemed too long for a background process. It looks like an oversight.

The lock holds a "current position pointer" during a multi-step embedding scan. The scan processes posts in batches across multiple requests. If the lock expires while the scan is running, the position pointer resets and the scan starts from the beginning.

Users publishing posts during the scan would restart the scan every four hours instead of completing it. The 24-hour lock was not a mistake. It was intentional protection for a long-running background operation.

**Long time-to-live values on locks are not automatically bugs. Read the function's purpose before assuming the value is wrong.**

## Error 3: Retry Count Without Checking the Interval

The AI changed a license redial limit from `< 10` to `< 30`:

```php
// Before (correct)
if ($redial_count < 10) {

// After (wrong)
if ($redial_count < 30) {
```

The reasoning was that 10 retries seemed low.

The license check fires on a schedule. Looking at `License.php:38`, the interval is `delta > 60*60*24*3`: every 3 days.

10 retries at 3 days per retry equals 30 days of grace period for a disconnected license. That is already a generous window.

Changing the limit to 30 creates a 90-day grace period. A user with a lapsed license gets three months of uninterrupted service.

The AI did not check the cron schedule. The number 10 looked low in isolation. In context, with the schedule factored in, the math was already correct.

**Always find the cron schedule or hook frequency before reasoning about retry counts, timeout values, or anything that depends on interval.**

## Error 4: Response Format Without Reading the Consumer

The AI changed a PHP function to unwrap an array from the JSON response:

```php
// Before
wp_send_json([$plan]);

// After
wp_send_json($plan);
```

The reasoning was that wrapping `$plan` in an array seemed unnecessary.

The function was hooked via `wp_ajax_wpil_start_maintenance`. Somewhere in the JavaScript, there was a consumer that read `data[0]` to get the plan. Changing the PHP output from an array to a scalar would have broken the JavaScript client silently: `data[0]` would now return the first character of a string instead of the plan object.

The CTO removed the entire function instead. It was old scaffolding from an earlier build that had never been connected to active code.

**Never change the shape of an API response without reading the consumer. The PHP side is half the contract.**

## The Common Root

All four errors share one cause: the AI analyzed each code site in isolation. It pattern-matched on what the code looked like at that specific line without tracing the full context.

Escaping looked missing. It was present upstream.
A lock duration looked too long. It was designed to be long.
A retry count looked too low. The interval made it correct.
An array wrapper looked unnecessary. The consumer required it.

Senior engineers do not read code in isolation. They trace data through transformations. They read the call chain. They check what interacts with what they are changing.

AI tools currently do not do this naturally unless you ask them to. The fix is to change how you prompt: do not ask "fix this function," ask "trace the data path through this function and tell me what state it is in before and after each transformation."

## What I Learned

**Trace data state before adding escaping.** Search for existing sanitization functions in the data path. The data you are about to escape may already be sanitized.

**Read lock purpose before changing duration.** Long locks are often protecting long operations.

**Find the cron interval before reasoning about retry counts.** Numbers that look wrong in isolation can be exactly right with the schedule factored in.

**Read the consumer before changing output shape.** PHP and JavaScript form a contract. Both sides need to match.

**Ask AI to trace, not just fix.** "Trace the data path and tell me its state at each step" produces better analysis than "find and fix bugs."

## Related

- [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](claude-appends-text-after-json.md) — another case where the bug was invisible until full context was examined
- [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](three-cascading-bugs.md) — three more failures that passed the build but broke in production
- [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](telegram-privacy-mode.md) — a platform-level silent failure
- [Telegram Bots Cannot DM Users Who Have Not Pressed Start](telegram-bots-cant-dm.md) — another class of invisible failure
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a rendering-layer failure with no error output
- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](support-tickets-root-causes.md) — the full bug sprint this review was part of
- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md) — what happened to the support infrastructure after the bug sprint: nobody was logging into the dashboard, so it was deleted and replaced with a cron digest

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

---

**→ Project:** [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) — the codebase where these AI-generated wrong fixes were caught 🔒

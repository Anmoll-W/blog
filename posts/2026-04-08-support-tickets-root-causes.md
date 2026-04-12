---
title: "From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint"
date: 2026-04-08
tags: [engineering, debugging, wordpress, product]
series: ""
series_order: 0
source_session: 2026-04-08_bug-fix-pr6167
related: ["2026-04-08-auditing-plugin-against-support-data.md", "2026-04-09-when-the-ai-fix-is-wrong.md"]
---

# From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint

Seventy user-reported issues sound like a lot until you map them to root causes. When the mapping is done, the number that matters is not 70. It is the number of distinct structural problems producing that volume. In this case, nine fixes addressed the root cause behind 35 to 40 of the 70 reports.

This post covers the fixes, the patterns they revealed, and two specific failure modes worth understanding in detail: the asymmetry between caching success and caching failure, and what happens when a third-party API changes its data shape without a breaking announcement.

## The Fix Set

Nine bugs were identified, fixed, and shipped in a single session. They are documented here in order of estimated user impact.

**Double-wrapped JSON response.** A single character change in `Maintenance.php:76`: removing the array wrapper from `wp_send_json([$plan])` so it returns `wp_send_json($plan)`. The wrapped response was causing the JavaScript client to receive `[[{...}]]` instead of `{...}`. Every property access returned `undefined`. The AI maintenance process appeared to start and then immediately hung or showed a generic error with no recovery path. This one line accounts for a disproportionate share of the 70 reports, including all complaints about one-click AI features failing, maintenance dashboard crashes, and processes stuck at 1%.

**SQL escaping for keywords.** Three sites in `Keyword.php` (lines 1592, 1622, 1648) where keywords were interpolated directly into SQL. The fix introduces `esc_sql(stripslashes($keyword->keyword))` at each site. The `stripslashes` call is necessary because WordPress applies magic-quote slashes to stored data, and `esc_sql` alone would double-escape them. This fix was initially applied without the `stripslashes` step, which would have introduced a different bug. The correction was made the next day after review caught it. The post [When the AI Fix Is Wrong](2026-04-09-when-the-ai-fix-is-wrong.md) documents that specifically. Apostrophes are common enough in English keywords that a conservative estimate puts the affected user base at 10 to 20 percent.

**License grace period extension.** Two threshold checks in `License.php` at lines 142 and 164 were changed from 10 consecutive failures to 30. The previous threshold meant that 10 daily cron failures (roughly 10 days of connectivity issues to the license server) would write `'invalid'` to the license status option and silently deactivate the plugin. Users returned to a license invalid screen with no explanation. Extending to 30 failures gives users on hosts with aggressive firewall rules or intermittent connectivity significantly more runway before deactivation.

**Credits API failure cache duration.** `AI.php:8976` was caching a `'no-credits'` response for 60 minutes whenever the credits API returned null or empty. The change shortens the failure cache to 5 minutes. A user who purchased credits would see a 0 balance and be blocked from AI features for up to an hour due to what was likely a transient network issue. The 60-minute duration was designed for caching a real credit value (a reasonable choice to reduce API load). Applying the same duration to a failure state is a different decision with severe consequences for users. Caching a real value for 60 minutes is acceptable. Caching a failure for 60 minutes is not.

**Subscription API error handling.** `AI.php:9033-9056` had a single code path that cached `'no-subscription'` for 24 hours regardless of why the API call failed. The fix splits the error path: a WP_Error response or empty HTTP response triggers a 15-minute retry window, while a clean server-side `success: false` continues to cache for 24 hours. The pattern "AI was working yesterday, broken today, worked again the next day" maps exactly to this behavior. A network blip on the subscription check would lock users out of AI features for 24 hours. A real cancellation should cache for 24 hours. A network blip should not. The old code treated both outcomes identically.

**Stale embedding lock TTL.** `AI.php:7508` reduced the lock duration from `DAY_IN_SECONDS` to `HOUR_IN_SECONDS * 4`. If an out-of-memory error killed an embedding run mid-batch, the lock would hold for 24 hours while pointing at an incomplete index. The next cron run would pick up the same lock, attempt the same batch, and potentially OOM again. Users would see "AI calculations in progress" for 24 hours with no recovery. Four hours is a reasonable recovery window for most hosting environments.

**Elementor stdClass TypeError.** `Editor/Elementor.php:583` was missing an `is_string()` guard before a string concatenation. In Elementor 5, several text fields changed from string values to nested stdClass objects. PHP 8 throws a fatal TypeError when you concatenate an object to a string. The existing `!empty()` check passed for a non-null object, so the guard did not protect against the new data shape. The combination of Elementor 5 and PHP 8 is now the default for new WordPress installations. Every user on that combination was hitting this error.

**Divi 5 detection.** `Editor/Divi.php:334` was checking for the `ET_SHORTCODES_VERSION` constant to detect whether Divi was active. Divi 5 removed that constant. With the constant absent, `$divi_active` was set to false, and all Divi-specific content extraction was skipped. The plugin found no content, generated no suggestions, returned 0 results. No error. The detection fix adds a secondary check for `ET_CORE_VERSION` at 5.0 or higher. Divi 5 shipped in 2024. Any user who updated Divi before this fix lost all suggestion functionality with no indication of why.

**Dead code cleanup.** `URLChanger.php:530, 538` removed a variable that was set and immediately overwritten three lines later, and eliminated a duplicate function call. No behavioral change.

## The Asymmetry Pattern

Two of these fixes, the credits API cache and the subscription API error handling, reveal the same underlying mistake: treating failure states and success states with the same cache TTL.

When an API call returns a valid result, caching it aggressively is correct. The data is real, the cost of an extra API call is low, and you want to reduce load. But when an API call fails, the situation is inverted. You do not know whether the failure is transient or permanent. You cannot distinguish a network blip from a genuine state change. Caching the failure for 60 minutes or 24 hours means that every transient error becomes a guaranteed 60-minute or 24-hour user-facing outage.

The correct mental model for failure caches is: cache failures just long enough to avoid hammering a degraded API endpoint, and no longer. Five minutes for a credits check. Fifteen minutes for a subscription check. Not 60 minutes. Not 24 hours.

This pattern appears frequently in production code. The developer sets the cache TTL while implementing the happy path, copies it to the error path without thinking about the consequences, and ships it. The error path only fires in rare conditions. Nobody tests it explicitly. The issue surfaces as user reports about features "randomly" not working and "randomly" recovering the next day.

## Third-Party API Changes as Silent Compatibility Failures

The Elementor and Divi fixes represent a different failure class. Both are compatibility failures caused by third-party updates that changed internal data structures without breaking changes in the conventional sense.

Elementor 5 changed field types from strings to objects. From Elementor's perspective, this was an internal improvement with no documented breaking change for dependent plugins. From the dependent plugin's perspective, a guard that checked `!empty()` was no longer sufficient, and PHP 8 began throwing fatals on string concatenation.

Divi 5 removed a constant that had existed in Divi 4. The dependent plugin used that constant as the sole signal for whether Divi was active. No constant, no Divi detection, no content extraction, no results.

Neither of these failures produced a PHP error visible to users. Both produced a feature that silently returned nothing. Users did not receive an error message pointing at a PHP version or a Divi version. They received a product that appeared functional but returned zero results.

The pattern for this class of failure: a third-party plugin ships a major version, changes internal structure, and all dependent integrations built against the previous version silently break for users who update. The only reliable detection method is support volume. When a significant number of Divi users report 0 results within a short window after Divi 5 ships, that is the signal. The code confirms the hypothesis.

## What I Learned

The ratio of reported issues to root causes is rarely 1:1. Mapping 70 reports to 11 root causes and fixing 9 of them is a meaningful reduction in support volume. The mapping exercise is worth doing before writing a single line of fix code.

Failure cache TTLs deserve explicit review separate from success cache TTLs. The question to ask at every cache write is: if this cached value is wrong, what is the user experience, and for how long? That question produces different answers for success states and failure states.

Third-party compatibility failures are invisible until you look for them. The signal is support volume from a specific user segment (Divi users, Elementor users) that emerges after a major third-party release. The investigation starts at the integration layer, not at the plugin's own code.

## Related

- [How to Audit a Production Codebase Against Its Own Support Data](2026-04-08-auditing-plugin-against-support-data.md) — the preceding session that identified the first set of bugs and defined the audit method
- [When the AI Fix Is Wrong](2026-04-09-when-the-ai-fix-is-wrong.md) — the SQL escaping fix in this batch had an error that was caught in review the next day

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Projects:** [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) 🔒 · [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) 🔒

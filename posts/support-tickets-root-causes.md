<!-- source_session: 2026-04-08_bug-fix-pr6167 -->

# From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint

The support backlog had 70 reported issues. The question on the table was what to fix first — and more specifically, whether fixing individual tickets one by one was the right strategy at all. If each report was its own independent bug, 70 fixes was the scope. If some fraction of those reports shared a root cause, the math changed significantly. Before committing to a fix strategy, the mapping had to happen. That mapping decision — do the root-cause analysis before writing any fix code — is what made this session produce nine shipping fixes in a single day instead of two or three.

In this case, nine fixes addressed the root cause behind 35 to 40 of the 70 reports.

This post covers the fixes, the patterns they revealed, and two specific failure modes worth understanding in detail: the asymmetry between caching success and caching failure, and what happens when a third-party API changes its data shape without a breaking announcement.

## The Fix Set

Nine bugs were identified, fixed, and shipped in a single session. They are documented here in order of estimated user impact.

**Double-wrapped JSON response.** A single character change in `Maintenance.php:76`: removing the array wrapper from `wp_send_json([$plan])` so it returns `wp_send_json($plan)`. The wrapped response was causing the JavaScript client to receive `[[{...}]]` instead of `{...}`. Every property access returned `undefined`. The AI maintenance process appeared to start and then immediately hung or showed a generic error with no recovery path. This one line accounts for a disproportionate share of the 70 reports, including all complaints about one-click AI features failing, maintenance dashboard crashes, and processes stuck at 1%.

**SQL escaping for keywords.** Three sites in `Keyword.php` (lines 1592, 1622, 1648) where keywords were interpolated directly into SQL. The fix introduces `esc_sql(stripslashes($keyword->keyword))` at each site. The `stripslashes` call is necessary because WordPress applies magic-quote slashes to stored data, and `esc_sql` alone would double-escape them. This fix was initially applied without the `stripslashes` step, which would have introduced a different bug. The correction was made the next day after review caught it. The post [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) documents that specifically. Apostrophes are common enough in English keywords that a conservative estimate puts the affected user base at 10 to 20 percent.

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

The product decision here is worth naming explicitly. The original 60-minute cache for the credits check was correct for its intended purpose: reduce load on the credits API. But that decision was made while implementing the success path. The error path inherited the same TTL without anyone asking the question: "if this value is wrong, what is the user experience, and for how long?" For a success cache, a wrong value means slightly stale data for 60 minutes. For a failure cache, a wrong value means a user who just purchased credits cannot access AI features for an hour with no explanation. Those are not the same decision. They were treated as one.

The correct mental model for failure caches is: cache failures just long enough to avoid hammering a degraded API endpoint, and no longer. Five minutes for a credits check. Fifteen minutes for a subscription check. Not 60 minutes. Not 24 hours.

This pattern appears frequently in production code. The developer sets the cache TTL while implementing the happy path, copies it to the error path without thinking about the consequences, and ships it. The error path only fires in rare conditions. Nobody tests it explicitly. The issue surfaces as user reports about features "randomly" not working and "randomly" recovering the next day.

The relatable version of this mistake: the cache TTL had been reviewed in prior work. The assumption was that caching was handled correctly because the success case had been thought through. The error path had not been reviewed separately because it felt like a subset of the same logic. It is not. It is a different decision that deserves its own review.

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

The decision lesson: the choice to map first and fix second is a product prioritization decision, not just an engineering one. It determines which users get relief and in what order. Fixing the 70 reports sequentially — triaging by recency or ease — would have produced different fixes in a different order, with less impact. Doing the mapping upfront meant the first nine fixes covered the root causes behind half the reported volume. Any time there is a support backlog of more than 10 reports, the right first step is the map, not the fix list. The map is the product strategy. The fixes are the execution.

---

**[2026-04-08](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![debugging](https://img.shields.io/badge/debugging-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![wordpress](https://img.shields.io/badge/wordpress-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [How to Audit a Production Codebase Against Its Own Support Data](auditing-plugin-against-support-data.md) — the preceding session that identified the first set of bugs and defined the audit method
- [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) — the SQL escaping fix in this batch had an error that was caught in review the next day
- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md) — what the support data ultimately informed: the category groupings that shaped the 7 AM digest that replaced the full dashboard
- [We Were Paying for a Support Tool. Gmail Already Did the Job.](gmail-support-system-without-paid-tools.md) — the ticket category taxonomy from this sprint directly informed the routing labels in the Gmail filter system that replaced the paid shared inbox

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Projects:** [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) 🔒 · [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) 🔒

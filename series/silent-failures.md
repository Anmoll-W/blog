# Series: Silent Failures

Bugs that do not throw obvious errors. They pass tests, they compile cleanly, and they fail in production when real inputs reveal the assumption that was wrong from the start.

This series documents specific cases from real projects. Each post covers one bug, why it was invisible in development, and the exact fix.

## Posts in This Series

1. [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](../posts/2026-04-04-claude-appends-text-after-json.md)
   When Claude adds explanatory text after a JSON response body, `JSON.parse` crashes at the trailing characters. The bug was invisible in early testing because Claude happened to return clean JSON. The fix: extract JSON by bracket position, never assume the full string is valid JSON.

2. [When the AI Fix is Wrong: What Senior Review Catches](../posts/2026-04-09-when-the-ai-fix-is-wrong.md)
   Three AI-generated fixes that introduced new bugs. All three had the same root cause: the AI analyzed code in isolation without tracing data lifecycle or checking math against intervals. Covers SQL double-escaping, lock TTL semantics, and retry count math.

3. [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](../posts/2026-03-29-three-cascading-bugs.md)
   Three distinct failures on a single deployed page, none of which threw a visible error. Module-level SDK initialization breaks at build time, not runtime. Multiple layout bugs can coexist and mask each other. A font rendering failure in Turbopack produces a blank section with no console output.

4. [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](../posts/2026-03-30-telegram-privacy-mode.md)
   A default Telegram bot setting that silently drops all non-slash-command messages in group chats. Also covers a state machine with missing cases and a date parser that only handled one format out of the many ways real users express dates.

5. [Telegram Bots Cannot DM Users Who Have Not Pressed Start](../posts/2026-04-04-telegram-bots-cant-dm.md)
   A platform constraint that blocks bots from initiating DMs to users who have not interacted directly with the bot. The API returns success. The message is never delivered. The entire preference collection flow had to be redesigned around in-group inline keyboards.

6. [Missing Viewport Tag: The Silent Root of All Mobile Failures](../posts/2026-04-10-missing-viewport-tag.md)
   A production site missing the viewport meta tag: every CSS breakpoint fires against a 980-pixel virtual viewport, not the device. The development environment injected the tag automatically, making the bug invisible until physical device testing. One line in `header.php` restored full mobile responsiveness.

## What Is Coming

- PHP magic quotes: when the data you are about to sanitize is already sanitized
- Third-party API shape changes: how Elementor 5 and Divi 5 silently broke dependent plugin integrations

---

*[All posts](../README.md)*

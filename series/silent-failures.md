# Series: Silent Failures

Bugs that do not throw obvious errors. They pass tests, they compile cleanly, and they fail in production when real inputs reveal the assumption that was wrong from the start.

This series documents specific cases from real projects. Each post covers one bug, why it was invisible in development, and the exact fix.

> **Related projects:** [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) 🔒 · [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) 🔒 · [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) 🔒 — the codebases where these bugs were found

## Posts in This Series

1. [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](../posts/claude-appends-text-after-json.md)
   When Claude adds explanatory text after a JSON response body, `JSON.parse` crashes at the trailing characters. The bug was invisible in early testing because Claude happened to return clean JSON. The fix: extract JSON by bracket position, never assume the full string is valid JSON.

2. [When the AI Fix is Wrong: What Senior Review Catches](../posts/when-the-ai-fix-is-wrong.md)
   Three AI-generated fixes that introduced new bugs. All three had the same root cause: the AI analyzed code in isolation without tracing data lifecycle or checking math against intervals. Covers SQL double-escaping, lock TTL semantics, and retry count math.

3. [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](../posts/three-cascading-bugs.md)
   Three distinct failures on a single deployed page, none of which threw a visible error. Module-level SDK initialization breaks at build time, not runtime. Multiple layout bugs can coexist and mask each other. A font rendering failure in Turbopack produces a blank section with no console output.

4. [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](../posts/telegram-privacy-mode.md)
   A default Telegram bot setting that silently drops all non-slash-command messages in group chats. Also covers a state machine with missing cases and a date parser that only handled one format out of the many ways real users express dates.

5. [Telegram Bots Cannot DM Users Who Have Not Pressed Start](../posts/telegram-bots-cant-dm.md)
   A platform constraint that blocks bots from initiating DMs to users who have not interacted directly with the bot. The API returns success. The message is never delivered. The entire preference collection flow had to be redesigned around in-group inline keyboards.

6. [Missing Viewport Tag: The Silent Root of All Mobile Failures](../posts/missing-viewport-tag.md)
   A production site missing the viewport meta tag: every CSS breakpoint fires against a 980-pixel virtual viewport, not the device. The development environment injected the tag automatically, making the bug invisible until physical device testing. One line in `header.php` restored full mobile responsiveness.

7. [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](../posts/launchd-icloud-silent-block.md)
   Scheduled vault automations that were registered with `launchctl list` but never actually running. launchd exec returned EPERM because the runner scripts lived inside an iCloud-synced folder. A second finding in the same audit: a plist that had been loaded pointing at a shell script that never existed. Fix required moving every runner out of iCloud and adding artifact checks, not just error checks.

8. [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](../posts/token-burn-audit.md)
   Five silent automation bugs in a LaunchAgent-based vault OS, each invisible because it was the absence of a signal rather than the presence of an error. A done-file contract that only accepted exit 0 turned one weekly job into 18 daily sessions. A Perl alarm timer that does not survive exec gave a 2am consolidator a seven-hour runtime. Two zombie agents whose self-disable logic ran after exec or against a path that had moved. And a BOOT context load that had grown to 180,000 characters with no compaction enabled.

## What Is Coming

- PHP magic quotes: when the data you are about to sanitize is already sanitized
- Third-party API shape changes: how Elementor 5 and Divi 5 silently broke dependent plugin integrations

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*

# Series: Silent Failures

Bugs that do not throw obvious errors. They pass tests, they compile cleanly, and they fail in production when real inputs reveal the assumption that was wrong from the start.

This series documents specific cases from real projects. Each post covers one bug, why it was invisible in development, and the exact fix.

## Posts in This Series

1. [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](../posts/2026-04-04-claude-appends-text-after-json.md)
   When Claude adds explanatory text after a JSON response body, `JSON.parse` crashes at the trailing characters. The bug was invisible in early testing because Claude happened to return clean JSON. The fix: extract JSON by bracket position, never assume the full string is valid JSON.

2. [When the AI Fix is Wrong: What Senior Review Catches](../posts/2026-04-09-when-the-ai-fix-is-wrong.md)
   Three AI-generated fixes that introduced new bugs. All three had the same root cause: the AI analyzed code in isolation without tracing data lifecycle or checking math against intervals. Covers SQL double-escaping, lock TTL semantics, and retry count math.

## What Is Coming

- Module-level SDK initialization in Next.js: why `new Bot(token)` at module scope breaks every build
- Font rendering failures: when a font weight silently renders as invisible text
- PHP magic quotes: when the data you are about to sanitize is already sanitized

---

*[All posts](../README.md)*

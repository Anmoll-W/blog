<!-- source_session: 2026-04-04_e2e-testing-and-json-fix -->

# Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites

*2026-04-04 · claude-api, json, debugging · Silent Failures*

> The bug did not appear in early testing. Claude happened to return clean JSON. Under load, it did not.

## Context

I had built the core experience of ChalotripBot around a promise: natural language in, structured trip plans out. The entire product relied on Claude returning clean JSON at 8 different points in the flow. I had never explicitly verified that assumption would hold at scale. On submission day it did not.

Running a full end-to-end test against a real Telegram group with real webhook payloads, every single itinerary generation call crashed with the same error:

```
SyntaxError: Unexpected non-whitespace character after JSON at position 10997
```

The fix took 30 minutes once I understood the cause. But finding it took longer, because the bug had been invisible during all earlier testing. I had shipped eight call sites on the assumption that "Claude returns JSON when asked" — and I had tested that assumption only with short, simple prompts that never gave Claude a reason to elaborate.

## What Was Happening

The Claude API is not a database. It is a language model. When you ask it to return JSON, it usually does. But "usually" is not "always."

Occasionally, Claude appends an explanatory note after the JSON body:

```
[{...day1...}, {...day2...}]
This itinerary is optimized for beach lovers and keeps within your budget.
The activities are sequenced to minimize travel time across each day.
```

The original cleanup code only stripped markdown fences:

```typescript
text.replace(/```json\n?|\n?```/g, "").trim()
```

That handles the case where Claude wraps JSON in a code block. It does not handle the case where Claude writes valid JSON and then keeps writing.

`JSON.parse` on the full string fails at position 10997 because position 10997 is where the closing bracket of the JSON array ends and the sentence "This itinerary is optimized..." begins.

## The Fix

The fix is to stop trusting that the entire string is JSON, and instead find the JSON by bracket position:

```typescript
function extractJson(text: string): string {
  const stripped = text.replace(/```json\n?|\n?```/g, "").trim();
  const start = stripped.search(/[\[{]/);
  if (start === -1) return stripped;
  const isArray = stripped[start] === "[";
  const end = isArray ? stripped.lastIndexOf("]") : stripped.lastIndexOf("}");
  if (end === -1) return stripped;
  return stripped.slice(start, end + 1);
}
```

This finds the first `[` or `{`, finds the matching closing bracket, and slices only that range. Everything outside the outermost brackets is discarded.

Applied to all 8 Claude parse sites in `claude.ts`.

## Why It Was Invisible in Early Testing

This is the part worth understanding.

In early testing, Claude returned clean JSON every time. The prompts were straightforward. The responses were short. Claude had no particular reason to add notes.

Under load, with complex trip parameters, longer context windows, and more data in the request, Claude started treating the response as an opportunity to be helpful. The explanatory text was Claude doing what it was trained to do: communicate clearly. It just happened to break a machine consumer that expected pure JSON.

The bug was not deterministic. It appeared inconsistently, which made it harder to catch in isolated unit tests or short development sessions. It showed up reliably only during full end-to-end testing with real production-scale inputs.

## The Testing Lesson

The session also uncovered a second issue with the test setup itself. The test script was marking responses as failures based on the response shape. But grammY's `webhookCallback` returns an empty 200 on success, not `{"ok": true}`. The test was scoring 30 successful webhook deliveries as failures because it expected a JSON body that grammY never sends.

This matters: your test pass or fail logic must account for what the framework actually returns, not what you expect a framework to return based on convention.

## What I Learned

**Never assume the entire API response string is valid JSON.** Always extract by bracket position. This applies to any language model API, not just Claude.

**Test with production-scale inputs, not simplified ones.** The bug only appeared when Claude had enough context to feel like explaining itself. Short inputs in early testing never triggered it.

**Verify test assertions against framework behavior.** An empty 200 from grammY is a success. A test that expects `{"ok": true}` and gets `""` will report a false failure.

**Apply the fix at every parse site, not just the one that triggered the error.** Once you understand the root cause, audit all similar patterns. There were 8 call sites. All 8 needed the fix.

**Validate your assumptions about external services before you build a product feature on top of them.** "Claude returns JSON when asked" is a reasonable assumption for simple prompts. It is not a tested contract. Any time a product flow depends on a probabilistic output — from a language model, an external API, or a third-party service — the assumption needs to be verified against production-scale inputs before the feature ships, not discovered broken on the day it matters.

---

**[2026-04-04](../README.md#all-posts)** · [![claude-api](https://img.shields.io/badge/claude--api-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![debugging](https://img.shields.io/badge/debugging-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![json](https://img.shields.io/badge/json-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![silent-bugs](https://img.shields.io/badge/silent--bugs-d73a4a?style=flat-square&logoColor=white)](../README.md#all-posts) [![typescript](https://img.shields.io/badge/typescript-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![ai-engineering](https://img.shields.io/badge/ai--engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Silent Failures →](../series/silent-failures.md)

## Related

- [When the AI Fix is Wrong: What Senior Review Catches](when-the-ai-fix-is-wrong.md) — a companion piece about the other side of AI-assisted coding: when the AI introduces bugs it cannot see
- [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](three-cascading-bugs.md) — three more bugs that looked fine until production
- [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](telegram-privacy-mode.md) — a platform-level silent failure in the same project
- [Telegram Bots Cannot DM Users Who Have Not Pressed Start](telegram-bots-cant-dm.md) — another class of silent failure from the same session
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a different rendering-layer silent failure

---

*Building in public from an Obsidian vault. I am Anmoll, a product manager who ships products using AI tools. [All posts](../README.md)*

---

**→ Project:** [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — the bot where this Claude API bug was found 🔒

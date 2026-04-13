---
title: "Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font"
date: 2026-03-29
tags: [engineering, silent-failures, nextjs, debugging]
series: silent-failures
series_order: 3
source_session: 2026-03-29_boarding-pass-redesign
related:
  - claude-appends-text-after-json.md
  - when-the-ai-fix-is-wrong.md
  - telegram-privacy-mode.md
  - telegram-bots-cant-dm.md
  - missing-viewport-tag.md
---

# Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font

The build succeeded. The page loaded. Nothing on the surface indicated anything was wrong.

That is the setup for all three bugs from a boarding pass redesign session. The page was a full-viewport layout reimagined as a boarding pass: left 62% with editorial display headlines and flight data, right 38% stub with monospace labels and a call-to-action button. After deploying, three distinct failures appeared in sequence. None of them threw a visible error. Each one required a different diagnostic lens.

## Bug 1: Module-Level SDK Breaks the Build

The error during `next build` was `supabaseUrl is required`. The line it pointed to was inside the API route file, but the actual problem was one level up.

`const supabase = createClient(...)` was written at module level, outside any handler function. This runs at build time. At build time, environment variables are not available. The Supabase client validates its credentials at construction, so it threw immediately.

The fix was moving `createClient(...)` inside the POST handler function, where it only runs at request time when environment variables are present.

This is the same class of error as initializing a Telegram bot with `new Bot(token)` at module scope. Any SDK that validates credentials at construction time will fail during a Next.js build if called outside a function. The error looks like a missing environment variable. It is not. The variable exists. The timing is wrong.

The pattern is worth internalizing: if a build error references a credential, check whether the SDK is being constructed at module level before checking whether the variable is actually missing.

## Bug 2: Horizontal Scroll With Three Simultaneous Causes

After fixing the build, the deployed page had a horizontal scroll. Tracking down one cause would have been straightforward. There were three, all active at once.

A ticker strip used `width: max-content`, which caused it to extend beyond its container. A perforation divider in the success state used a negative margin that pushed content outside the page boundary. The boarding pass container had `overflow: visible`, which was intentional for decorative punch hole elements but had the side effect of allowing both bleeds to escape.

Fixing one at a time would have produced inconsistent results. The fix required addressing all three:

- `html, body { overflow-x: hidden; max-width: 100% }` as a root-level catch
- `.bp-pass { overflow: hidden }` to contain the ticker strip
- Removing the negative margin and removing the punch hole divider elements that required `overflow: visible` in the first place

The decision here was clear. A horizontal scroll on a mobile page is a hard failure. Decorative punch holes are a nice visual detail. When these two requirements conflict, the scroll fix wins. The punch holes were removed.

The broader lesson is that when multiple layout bugs coexist, fixing the most obvious one first can mask the others or make them harder to isolate. It is worth pausing to enumerate all the causes before writing any fix.

## Bug 3: Font Rendering Failure, No Error

The entire left section of the boarding pass was blank. The right section rendered normally. No console error. No layout shift. The DOM contained the text, the CSS was correct, and the color values were not in question.

The diagnostic signal was a faint label using IBM Plex Mono that was visible in the right section, versus a headline using Playfair Display at weight 800 italic in the left section that was completely invisible. Both used the same color. One rendered, one did not. This pointed to font rendering failure rather than contrast or color.

Swapping `Playfair_Display` for `Bodoni_Moda` resolved it immediately. Same editorial weight, confirmed working in the Next.js 16 and Turbopack build pipeline.

The specific failure appears to be Playfair Display at weight 800 italic not rendering correctly in Turbopack. There is no error surfaced anywhere in the build or runtime output. The text exists. It is simply invisible.

This is the hardest category of bug to debug because there is nothing to react to. The page appears healthy. The symptoms look like a content problem or a CSS specificity issue before you realize the font itself is silently failing.

## What I Learned

The build succeeding is not evidence that the page works. In all three cases, the failure was downstream of a clean build, and none of the failures produced the kind of error that triggers a normal debugging reflex.

When debugging layout issues in Next.js, the overflow properties at every level of the component tree matter. One `overflow: visible` on a parent can expose bleed from multiple children simultaneously.

Font rendering failures in the Turbopack pipeline are silent. If a section of a page is blank with no console error and the DOM is correct, test a font swap before investigating CSS.

## Related

- [Claude Appends Text After JSON](claude-appends-text-after-json.md) — another silent failure pattern: output that looks correct but breaks downstream parsing
- [When the AI Fix Is Wrong](when-the-ai-fix-is-wrong.md) — what happens when the suggested fix addresses the symptom, not the cause
- [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](telegram-privacy-mode.md) — a platform-level silent failure from the same session context
- [Telegram Bots Cannot DM Users Who Have Not Pressed Start](telegram-bots-cant-dm.md) — another invisible failure class in the same project
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a rendering-layer failure that produced zero error output

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

---

**→ Project:** [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — the dashboard where these three cascading bugs appeared 🔒

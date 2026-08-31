# CLAUDE.md — Learning in Public Blog

Rules for maintaining this blog. Follow all of these every time a post is published or the repo is updated.

---

## Repo Structure

```
blog/
├── README.md          ← full post index, always current
├── CLAUDE.md          ← this file
├── posts/             ← all posts, slug-only filenames
└── series/            ← series index files
```

---

## Rules — Always Follow When Publishing a Post

### 1. Filename format
- Always: `slug-only.md` — no date prefix
- Wrong: `2026-04-13-the-eval-agent.md`
- Right: `the-eval-agent.md`

### 2. Blog README update (required on every publish)
- Add new post to the `## All Posts` table in `README.md`
- Position: newest first
- Format: `| YYYY-MM-DD | [Full Post Title](posts/slug.md) | Series or — | tag1, tag2 |`
- Never leave a post out of the index

### 3. Post metadata line (required on every post)
- No YAML frontmatter. No `---` blocks.
- Instead, add a single italic line directly below the `# Title`:
  - With series: `*YYYY-MM-DD · tag1, tag2, tag3 · Series Name*`
  - Without series: `*YYYY-MM-DD · tag1, tag2, tag3*`
- source_session goes in an HTML comment above the title: `<!-- source_session: ... -->`
- Example:
  ```
  <!-- source_session: 2026-04-16_my-session -->

  # My Post Title

  *2026-04-16 · vault, ai, systems · Vault as OS*

  Content starts here...
  ```

### 4. Series file update (if post belongs to a series)
- Add entry to the relevant `series/*.md` file
- Include a 1-2 sentence description of what the post covers

### 4. GitHub profile README (`Anmoll-W/Anmoll-W` repo) — auto-push on every blog publish
- Always shows exactly 5 posts — the most recent by date
- Update on every publish: remove oldest post from the table, add new post at top
- Link format: full GitHub URL with slug-only path
  - `https://github.com/Anmoll-W/blog/blob/main/posts/slug.md`
- Profile README lives at: `/Users/aw/Projects/Anmoll07/README.md`
- **Auto-push rule (no manual nudge):** the moment the blog `git push` succeeds, run the same commit-and-push sequence in `/Users/aw/Projects/Anmoll07/`:
  1. `cd /Users/aw/Projects/Anmoll07 && git diff --stat README.md` — confirm the only diff is the top-5 row swap
  2. `git add README.md && git commit -m "Profile: surface <slug>" && git push`
  3. Report both commit SHAs (blog + profile) in the publish summary
- Treat blog publish and profile publish as one atomic action — never push the blog and stop. Stopping leaves the profile stale (the page that gets the most LinkedIn / Google traffic). If the profile diff is dirty beyond the expected row swap, stop and ask before committing — do not skip the push.

### 5. Cross-linking — mandatory on every publish
When a new post is published:

**Step A — link FROM the new post to existing posts:**
- Read the new post's topic, series, and tags
- Identify 2-5 existing posts that are topically related
- Add them to the `## Related` section at the bottom of the new post
**Step B — link TO the new post from existing posts:**
- For each related post identified in Step A, read its `## Related` section
- If the new post is relevant to that existing post's reader, add a link back
- Update the `## Related` section body only
- Prioritise: posts in the same series, posts covering the same system, posts that are the "before" to this post's "after"

**Cross-link quality bar:**
- Every link must have a description clause that explains WHY the reader should follow it
- Format: `[Title](slug.md) — one sentence on what connects these posts`
- Never add a link without the description clause

### 6. Internal link format
- Always relative paths from the post's location: `slug.md` (same directory) or `../README.md`
- Never absolute GitHub URLs inside post bodies
- Never date-prefixed paths

---

## Documentation Standard

All docs in this project follow `~/.claude/rules-available/doc-standard.md`.

Every spec, plan, PRD, and architecture doc must include:
- Doc type label at top (`**Doc type:** Architecture / Process / Spec / Reference / Explanation / Plan`)
- Audience named in one sentence
- At least one Mermaid diagram (flowchart, sequenceDiagram, stateDiagram-v2, erDiagram, or quadrantChart)
- Architecture docs: context diagram + container diagram, both required
- Glossary for any domain terms introduced

---

## Quality Bar — Every Post Before Pushing

Run a quick check against these before every commit:

- [ ] Filename is slug-only (no date)
- [ ] **Human writing standard (TWV-1392)** post follows `/Users/aw/Library/Mobile Documents/com~apple~CloudDocs/Obsidian/Aw Vault/Knowledge/human-writing-standard.md` (HAVE/AVOID card at its top): no Tier 1 AI vocabulary, one sentence under 6 words and one over 25, a concrete anchor early, at least 3 true human markers, plain verbs. GitHub blog carries no AI detector, so the reader is the only judge.
- [ ] No YAML frontmatter — metadata is the italic line below the title: `*YYYY-MM-DD · tags · Series*`
- [ ] source_session is in an HTML comment above the title
- [ ] Post has a `## Related` section with at least 2 links
- [ ] Blog README updated
- [ ] Series file updated (if applicable)
- [ ] Profile README updated (latest 5 posts)
- [ ] Existing posts updated with backlinks where relevant
- [ ] All facts are exact — no fabricated durations, scale claims, or outcomes (polish writing, never facts)
- [ ] **FACT SOURCES verified** — every number, count, and named entity in the draft traces to a file:line or live grep checked in this session. Plan files are forbidden as the source — they describe pre-execution intent, not what actually happened. Source-of-truth hierarchy: live grep > session logs > eval reports > decisions.md. Spec lives in `Knowledge/personas/_brain-protocol.md` § FACT SOURCES.
- [ ] **Pre-push numeric scan run** — `grep -nE '\b(one|two|three|four|five|six|seven|eight|nine|ten|eleven|twelve|thirteen|fourteen|fifteen|sixteen|seventeen|eighteen|nineteen|twenty|thirty|forty|fifty|sixty|seventy|eighty|ninety|hundred|thousand|[0-9]+)\b' posts/<slug>.md` — every match traces to a FACT SOURCES row, a date, or a colloquial usage. Unmatched numbers block the push. (This + FACT SOURCES is the full numeric gate — the external gemini-review step was retired 2026-06-29, removed from this checklist 2026-07-04.)

---

## Thought Process — Why These Rules Exist

**Why slug-only filenames?**
The GitHub file browser shows raw filenames. `the-eval-agent.md` reads like a post. `2026-04-13-the-eval-agent.md` reads like a dev log. The date prefix adds no value — it is already in the metadata line and the README table.

**Why always update the profile README?**
The profile README is the first thing someone sees when they click through from LinkedIn or a Google result. Stale posts signal an abandoned project. Keeping it to exactly 5 posts forces curation — not just appending.

**Why mandatory cross-linking?**
A blog where no posts know about each other is a dump. Cross-linking builds a reading path: someone who finds one post should be able to find everything related without going back to the index. It also signals to search engines that the content is connected and maintained.

**Why bidirectional links?**
Unidirectional links mean old posts never benefit from new posts. Every time a new post is published, it is usually the "next chapter" of an older post's story. That older post's reader deserves to know the story continued.

---

## Editing Discipline

Global coding/content discipline lives in `~/.claude/CLAUDE.md`. Project-specific:

- Don't reformat existing posts when publishing a new one — match their voice, leave them alone
- Don't restructure README — append to the table, don't reorder columns or change format
- Don't "improve" old posts' cross-links beyond the additions Step B requires
- Don't anonymize names retroactively in old posts — only enforce on new ones
- Verify before declaring publish done: `ls posts/*.md | wc -l` matches the row count in README's `## All Posts` table; profile README still shows exactly 5 entries

<!-- source_session: 2026-05-31_hermes-wave-1 -->

# Hermes, Wave 1: Giving My Vault an Always-On Body

*2026-05-31 · ai, systems, automation, vault · Hermes as a PA*

For about a year, my entire working life has lived in one place: a folder of Markdown notes on my Mac, edited inside Obsidian. Projects, decisions, finances, health, the running log of what I did and why — all of it is plain text in that vault. It is the single source of truth, and I trust it more than any app I have ever used.

But a folder on a Mac has one obvious limitation. It only does anything when I am sitting in front of it. It cannot notice that a task has been waiting on someone for two weeks. It cannot remind me of something at 5pm tomorrow if my laptop is closed. The brain was there. The body was not.

This is the first post in a new series about fixing that. The project is called Hermes, and Wave 1 — the part I am writing about here — is the unglamorous foundation underneath everything else. There are no clever features in most of it. It is plumbing, safety rails, and alarms. But it is the part that makes everything that comes later safe to build.

---

## The Three Layers

The system has three layers, and it helps to name them before going further.

**Layer one is the brain.** That is the Obsidian vault — the Markdown notes on my Mac. It holds the truth and it never moves off the machine wholesale.

**Layer two is the hands.** That is Claude Code, the interactive coding tool I use to build and edit. When I sit down to write or change something, I am working through the hands. They talk directly to Anthropic's models.

**Layer three is the body.** That is Hermes — an agent (built on NousResearch's Hermes) that runs all day and all night on a small rented Linux server, a Hetzner virtual private server. A virtual private server, or VPS, is just a computer you rent in a data centre; mine is a cheap one. Hermes is the ambient layer: a personal assistant you can text on Telegram, plus a set of background jobs that run on a schedule. Hermes routes its model calls through OpenRouter, a service that forwards requests to different model providers. It never touches Anthropic directly. That separation is deliberate, and I will come back to why.

The brain and the hands already existed before this project. Wave 1 is about giving them a body — something that can act on my behalf when I am nowhere near the keyboard.

---

## Why the Boring Foundation Comes First

Before any assistant that can read your notes does a single useful thing, it needs three properties, and none of them are features you can see.

It needs a **safe data boundary**, so that putting part of your brain on a rented server does not leak the parts that must never leave your own machine. It needs **jobs that fail loudly**, because a background task that quietly stops working is worse than no task at all — you come to depend on it and never learn it died. And it needs **a way to prove it is still alive**, so that silence can be trusted to mean "running," not "crashed three days ago."

Wave 1 builds exactly those three things, and then — only then — the first two features you can actually feel.

The method mattered as much as the goal. I sliced the work into fourteen small units I called micro-waves. Each one solved exactly one problem completely before the next one started. No bundling, no "I'll come back to that." A unit that left a gap went back to the start; it never moved forward.

And every micro-wave was reviewed by two adversarial AI personas whose entire job was to break it. One, Vera, plays quality and testing. The other, Alex, plays platform engineering. They are prompted to be hostile — to find the leak, the race condition, the silent failure I did not think of.

That setup produced one rule that turned out to matter more than any other, so I will state it plainly here and show it in action later: **validate every finding before you fix it.** The reviewers were sometimes wrong. A finding flagged as critical would sometimes evaporate the moment I tried to reproduce it for real. So nothing got changed on the strength of a review alone. Every reported bug had to be reproduced against the actual artifact first. Real ones got fixed. Phantom ones got dropped — and that discipline saved me from breaking working code to chase problems that did not exist.

---

## Problem 1 — A Boundary You Cannot Accidentally Cross

The hardest question came first: how do you give a rented server part of your brain without leaking the private parts?

My vault holds finance notes, health notes, and personal notes that must never, under any circumstances, leave the Mac. The server only needs a thin slice — the project notes, the daily logs, the task lists. So the rule became: sync only an explicitly named slice, and default everything else to "do not sync."

The tool for the sync is Syncthing, which keeps folders in step across two machines, peer to peer, with no cloud in the middle. Syncthing reads an ignore file that decides what crosses and what stays put. The safe way to write that file is deny-by-default: the first line says "ignore everything," and then you add back only the specific, named, safe folders. Nothing reaches the server unless I have explicitly allowed it.

The first version of that allowlist looked correct and was not. A reviewer caught it. The pattern I had written would have correctly blocked the sensitive folders that existed *today* — but if I created a new sensitive folder next month, it would have silently synced. The boundary protected the present and quietly betrayed the future. That is the worst kind of bug, because it passes every test you write on the day you write it.

I did not take the reviewer's word for it. I installed a real copy of Syncthing and proved the leak with the actual software, not a simulation of how I assumed it behaved. It was real. The fix was a stricter pattern that lists safe paths by type rather than waving a broad rule through, so a brand-new folder is ignored by default until I name it.

There is a second guard behind the first. Before anything syncs, a secret scanner reads the files looking for anything shaped like a key or a password. It is built to fail closed: if it cannot read a file, or it finds a secret-shaped string, it blocks the sync rather than assuming the file is safe. "I could not check this" is treated as "do not send this," never as "probably fine."

The verified result, checked on the live server: it holds roughly 140 notes and zero finance, health, or personal files. The only credential that does any model work on the server is the OpenRouter key. There are no other model-provider keys on the box at all — which is exactly why Hermes routes through OpenRouter and never Anthropic. The server simply does not hold the keys to do anything else.

---

## Problem 2 — One Writer Per File

The second problem is quieter and nastier: how do you avoid corrupting your own data when two machines can edit the same files?

Syncthing has a firm rule of its own. It never merges. If two devices change the same file before they sync, it does not try to reconcile them — it keeps both and leaves you a conflict copy to sort out by hand. Do that across hundreds of notes and you have a mess that is genuinely hard to untangle.

The defense is a discipline, not a tool: every file has exactly one device allowed to write it. The server owns some files; the Mac owns the rest; nothing is owned by both.

That sounds simple until you find the trap. On the Mac, a background indexer runs across the whole vault and rewrites the metadata at the top of every note. Harmless for years — except now some of those notes are owned by the server. The indexer would have rewritten files the server was also writing, and produced exactly the conflicts the single-writer rule exists to prevent. The fix was to teach the indexer to skip every path the server owns. It now leaves those files completely alone.

---

## Problem 3 — Nothing Fails Silently

A background job you cannot see is a job you cannot trust. So the third problem was making every scheduled task fail loudly, on purpose.

The answer is a small wrapper — I call it hermes-wrap — that every scheduled job runs inside. It does a few unglamorous things very firmly. It stops on the first error instead of plowing ahead. It kills a job that hangs, so a stuck task cannot run forever. And the moment a job fails, it sends a message to my phone over Telegram. The principle is that silence must never be mistaken for success. If something breaks, I find out immediately, not when I eventually notice the results stopped arriving.

Two more guards live in the same wrapper. A spending guard adds up the running cost of a job's model calls and kills it the instant it would cross a budget ceiling, so a runaway loop cannot quietly burn money overnight. And the logs are scrubbed of anything secret-shaped before they are written to disk, so the record of what happened never becomes a place a key could leak.

---

## Problem 4 — Proving It Is Alive

The fourth problem is the one most people skip, and it is the one I am proudest of. How do you know the assistant is still running when you are not looking?

The naive approach is to have the server send a "still alive" message on a schedule. The flaw is obvious once you see it: if the server is truly down, the part that would send the "I am down" message is also down. The alarm dies with the thing it was meant to watch.

The correct pattern inverts it. The server pings an outside service — I use Healthchecks.io, which on its free tier gives you a handful of checks and emails you when something goes wrong. The trick is that the server only ever sends "I am fine" pings. It never sends a failure. Instead, the outside service watches for the pings to *stop*. When they go quiet, it emails me. Silence itself is the alarm. The watchdog cannot fail to report a failure, because reporting is not its job — staying quiet is the failure.

There are two of these. One checks, every couple of minutes, that the assistant's service is actually running and that Telegram answers when prodded — not just that the process exists, but that the bot can really respond. The other confirms, every few minutes, that the server itself is up. If either falls silent, an email arrives. Nothing inside the server has to notice its own death.

---

## The Payoff — Two Features You Can Feel

All of that is invisible. Here is the part I actually use.

**A morning digest.** Every morning, the assistant reads its slice of the vault and sends one Telegram message. Not a dump of everything — a short, honest list. The handful of tasks I could genuinely finish today, and any delegations that have gone stale and need chasing. If I handed something off two weeks ago and it has not moved, the digest surfaces it before it rots. One message, the real picture, before I have opened the laptop.

**A reminder loop.** I text the bot something like "remind me to renew the domain tomorrow at 5pm." It parses the time, writes the reminder into the vault — which then syncs back to the Mac, so the reminder lives in the same brain as everything else — and replies "Reminder saved." A scheduled job resurfaces reminders that are due or overdue, so nothing falls through.

One detail in that flow matters more than it looks. The confirmation is only sent *after* the reminder has been durably saved. The "saved" message is a promise, not a hope. If you get the acknowledgement, the reminder is really on disk. I never wanted a bot that says "got it" and then loses the note.

---

## The Method That Made It Trustworthy

I said earlier that "validate before you fix" was the rule that mattered most. The reminder feature is where it earned its keep.

The first version of the reminder parser had a bug: it crashed on impossible dates. Tell it to remind you on "February 30th," or at "hour 25," and instead of handling the nonsense gracefully it fell over. A crash there is not a small thing — it is the assistant choking on input a human could obviously make. The adversarial review caught it. I reproduced it for real, confirmed it was genuine, and fixed it: the parser now degrades politely, asking for a clearer date instead of crashing.

That is the whole loop in one example. A hostile reviewer pokes at the thing. I reproduce the finding before believing it. The real ones get fixed; I do not thrash working code over the imaginary ones.

One more piece of honesty about how the reminder capture is wired. Hermes is a packaged agent — something I run, not something I rewrote. So rather than editing the agent's guts to catch reminders, I hung the capture off its own hook system: an observer that watches messages go by and cannot break the normal chat if it misbehaves. It is the safe place to add behaviour. The tradeoff is that the bot still replies conversationally to a reminder message on top of saving it, instead of giving one clean combined reply. I know about it. A tidier version is deliberately left for Wave 2 — it touches the live message path, and I would rather let the current setup soak before changing something that handles every message.

---

## What's Next

Wave 1 is the floor, not the house. With the boundary, the single-writer discipline, the fail-loud harness, and the watchdogs all in place, the next wave can build on solid ground.

Wave 2 will move my existing scheduled routines onto this hardened foundation and polish the reminder experience into that single clean reply. Each future wave will get its own post in this series — the same pattern of one problem at a time, broken on purpose by a hostile reviewer before I trust it.

The vision was never a clever assistant. It was a brain I already trusted, finally given a body that could act while I was away from the desk. Wave 1 is that body learning to stand up safely. The interesting part comes next.

---

## Related

- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — the next wave, where the same two-reviewer discipline caught five defects that green tests completely missed; the "after" to this post's foundation
- [Three Claude Tools, One Vault: The Architecture Behind the System](three-claude-tools-one-vault.md) — the brain-and-hands architecture this post finally gives a body to
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the scheduled-automation layer that Wave 2 will migrate onto this hardened foundation; the "before" to this post's "after"
- [Two Models, One Pipeline: Building a Multi-LLM Peer Review Gate That Caught Its Own Silent Bug](multi-llm-peer-review-gate.md) — the same adversarial-review posture (hostile personas, validate before you fix) that hardened every micro-wave here
- [How I Taught My Vault to Read YouTube](youtube-to-vault-pipeline.md) — a sibling build in the same voice, and the post where Hermes and OpenRouter first show up as the ambient layer
- [Why I Shut Down Hermes — a Multi-Agent AI System I Built Myself](why-i-shut-down-hermes.md) — the shutdown post; the data boundary and single-writer rule from this post held up, but they were not enough to offset the total maintenance overhead of the distributed layer built on top of them

---

*This post was distilled from a working session building the Hermes ambient layer. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES
- "three layers" (vault brain / Claude Code hands / Hermes body) — Projects/Hermes/CLAUDE.md "Mental model"; index.md
- "fourteen micro-waves" — vault execution plan §6 (MW-1..MW-14), all DONE this session
- "two adversarial personas" Vera + Alex — run via Workflow panels every micro-wave this session
- "roughly 140 notes / zero finance-health-personal / only OpenRouter key" — live VPS verify this session (find /root/hermes-slice, grep ~/.hermes/.env names)
- allowlist would-leak-future-folders + real-Syncthing proof — MW-1 log; slice/validate.sh 49/49
- indexer skips server-owned paths — MW-8; run_vault_indexer should_skip + tests (65 pass)
- hermes-wrap fail-loud + spend guard + log scrub — MW-3/4/5
- two watchdogs, Healthchecks free tier 3 checks email, dead-man, gateway 2min / VPS 5min — MW-9 live verify this session
- morning digest + reminder loop (ACK-after-save) + impossible-date crash→graceful — MW-10/11/12; reminder_store tests; live end-to-end verified this session
- capture via Hermes hook (observer) not package edit; Wave-2 single-reply deferral — MW-11 log
-->


<!-- source_session: 2026-06-02_hermes-how-it-works -->

# How My Hermes Agent Works — and What It Does for Me, From a PM's Point of View

*2026-06-02 · ai, systems, automation, vault · Hermes as a PA*

I have been writing a series about building a personal AI assistant called Hermes, one wave at a time. Those posts go deep on each piece as it ships. This one steps back and answers the question a product manager would actually ask first: what problem does this solve, how is it shaped, and what does it do for me on an ordinary day?

I am a product manager by trade, so I cannot help but look at my own side project the way I would look at any product. Where is the real problem? What did I decide to build, what did I decide *not* to build, and why? What constraints are non-negotiable, and what got sequenced for later? This post is that lens turned on Hermes. If you have read the wave posts, this is the map that sits above them. If you have not, this is the place to start.

---

## The Problem: A Brain With No Body

For about a year my entire working life has lived in one place — a folder of Markdown notes on my Mac, edited in Obsidian. Projects, decisions, finances, health, the running log of what I did and why. It is plain text, it is the single source of truth, and I trust it more than any app I have used.

But a folder on a Mac has one structural limitation, and it is the kind of limitation a PM should notice before writing a line of code. **It only does anything when I am sitting in front of it.** It cannot notice that a task has been waiting on someone for two weeks. It cannot remind me of something at 5pm tomorrow if my laptop is closed.

And here is the sharper version of that problem, the one that actually pushed me to build. I had tried automating some of this already, with jobs scheduled on the Mac itself. The trouble is that **a Mac sleeps.** A task scheduled for 2am on a machine that is asleep does not run — it silently does not run, and you find out late, or never. That is the worst failure mode a system can have: not "it broke and told me," but "it stopped and pretended nothing was wrong." Automation you cannot trust to actually fire is worse than no automation, because you come to depend on it.

So the real problem was never "I want a clever assistant." It was: **I have a brain I trust, and it needs a body that stays awake.** Something always-on, that I can also trust not to overstep.

---

## The Product Decision: Three Layers, One Job Each

The first real design decision was a separation-of-concerns call, and it is the one I would defend hardest. The system is three layers, and each layer has exactly one job.

**The brain** is the Obsidian vault — the Markdown notes. It holds the truth. It is just text, owned by no particular tool.

**The hands** are Claude Code, the interactive coding tool I use to build and edit. When I sit down to make or change something, I work through the hands. They talk directly to Anthropic's models.

**The body** is Hermes — an agent (built on NousResearch's Hermes) that runs day and night on a small rented Linux server, what is called a virtual private server, or VPS. A VPS is just a computer you rent in a data centre; mine is a cheap one, about six dollars a month. Hermes is the ambient layer: a personal assistant I can text on Telegram, plus background jobs that run on a schedule. It routes its model calls through OpenRouter — a service that forwards requests to different model providers — and never touches Anthropic directly.

Why split it this way instead of building one monolith that does everything? Because each layer fails differently and changes differently. The brain must stay tool-agnostic, so it outlives any single app. The hands are interactive and powerful and therefore should never run unattended. The body must be always-on and therefore must be the *least* powerful of the three — kept on a tight leash precisely because it acts while I am not watching. Folding those into one thing would have meant the always-on part inherited the powers of the interactive part, which is exactly the combination you do not want running while you sleep. The boundaries are the product.

There is one invariant that holds the whole thing together, and I treat it as load-bearing: **the vault is Hermes's live mirror.** Anything Hermes does flows back into the vault automatically, as a file Hermes alone owns. So when I open the hands in the morning, they see a complete, current picture. Hermes never holds state the brain cannot see. The corollary is a strict rule — every file has exactly one device allowed to write it. Two writers on one file is the one thing the underlying sync can never reconcile, so the design forbids it outright.

---

## Constraints as a Feature, Not an Afterthought

Most "guardrails" get bolted on after a product works. I wrote mine first, because for an agent that acts on my behalf the constraints *are* the spec. The headline one is a single sentence:

**Hermes never publishes, sends, trades, moves money, or changes my investments without me.**

It observes, it drafts, it alerts. I decide and approve. That is not a tone-of-voice promise; it is enforced in code. The content drafter, for example, has no posting capability at all — not a disabled one, an absent one. There is no function in it that can reach LinkedIn. The finance pieces can read and summarise but have no path that places a trade or touches my monthly investment. If a future, lazier version of me ever tries to wire one in, a test goes red.

A PM reflex worth naming here: **design for reversibility.** Every action Hermes can take is one I can undo or that costs nothing to ignore — a draft I delete, an alert I dismiss, a reminder I clear. Nothing it does on its own is one-way. The irreversible actions — publishing, paying, trading — are exactly the ones it is forbidden from taking. That mapping is not a coincidence. The line between "Hermes can do this alone" and "this needs me" is drawn precisely along reversibility. That is the safest place to draw it, and drawing it deliberately is the difference between an assistant I trust with my notes and one I would be nervous to leave running.

There is a privacy constraint of the same kind. A trimmed copy of the *non-sensitive* vault syncs to the server — project notes, daily logs, task lists. Finance, health, and personal notes never leave the Mac. The sync rule is deny-by-default: the boundary ignores everything and then names only the safe folders, so a new sensitive folder I create next month is excluded automatically rather than leaking by omission. And every model call that carries my notes is pinned to a paid, no-log route, because a free model route can quietly turn your prompt into someone's training data. When the prompt is a slice of your own brain, "free" is the most expensive option there is.

---

## The Roadmap: Ship Value Early, Pull Heavy Infra in Late

The sequencing was a deliberate product call, and it is the part I would put in front of any PM as a worked example. I had two ways to build this.

The tempting way was infra-first: stand up the full platform — containers, an orchestration layer, a complete sync, a memory store — and *then* build features on top. That is how a lot of ambitious side projects die. You spend months on plumbing, ship nothing you can feel, and lose the thread.

The way I actually shipped was **value-first, in waves.** Each wave delivers something I can use, and heavy infrastructure is pulled in only when a wave genuinely needs it.

- **Wave 1 — the MVP.** The unglamorous foundation: the safe data boundary, the single-writer rule, a harness that makes every job fail loudly, and watchdogs that prove the system is alive. Then exactly two features I can feel — a morning digest and a reminder loop. This is live.
- **Wave 2 — reliability and consolidation.** Move my scattered existing automation onto the hardened server and collapse a sprawl of little jobs into a handful of well-behaved ones. Live.
- **Wave 3 — content.** Teach the body to draft LinkedIn posts from the signals already in my vault, onto a bench of raw candidates I edit and post by hand. Live.
- **Wave 4 — finance and work intel.** A daily live snapshot of my long-term savings progress in the morning briefing, market alerts, and the ability to text it an investment to log. **Built, reviewed, and as of today deployed** — the jobs are on the server and the Mac producer is computing the figure. What it does not have yet is a soak; it has only just started running, so there are no outcomes to report.

The ordering is the lesson. Each wave shipped value before the next began, and no wave paid for infrastructure a later wave needed until that later wave arrived. A morning digest does not need containers, so Wave 1 did not build them. That is the opposite of how I am naturally tempted to work — I lean toward "do more" — and writing the roadmap as a sequence of shippable increments is what kept the project honest.

---

## What It Actually Does With Me

Here is the part that earns its keep on an ordinary day. None of it is autonomous; all of it is a draft, an alert, or a surfaced fact that I act on.

**Every morning, a briefing.** One Telegram message: the handful of tasks I could genuinely finish today, pulled from where they actually live in my vault, plus any delegations that have gone stale and need chasing. As of the latest wave it also carries one line on my long-term savings — a single progress figure and whether I am on track — computed privately and delivered without any account detail crossing to the server. One message, the real picture, before I open the laptop.

**Reminders I text it.** "Remind me to renew the domain tomorrow at 5pm." It parses the time, writes the reminder into the vault, and only then confirms it is saved — the acknowledgement is a promise the note is really on disk, not a hope. The briefing resurfaces anything due or overdue.

**Investments I text it (newly live).** "Invested fifty thousand in an index fund" gets logged to my finance notes as a record and a signal. Crucially, it logs — it does not invest. And it refuses to store anything that looks like a one-time password or a secret.

**Nightly vault maintenance.** While I sleep, it processes my inbox, consolidates loose notes, and keeps indexes consistent — the housekeeping I would otherwise never get to.

**LinkedIn drafting.** A couple of times a week it reads the ideas already sitting in my notes and drafts posts onto a bench, as raw candidates for two surfaces. It cannot post. I edit raw to ready and publish by hand.

**A weekly review and periodic alerts.** A Sunday review, and event-driven nudges — a market move past a threshold, a stale piece of work — all alert-only.

The common thread, and the thing I am most deliberate about: every one of these stops one step short of acting. It hands me the draft, the number, or the nudge. The decision stays mine.

---

## The PM Takeaways

Four things I would carry from this into any product, not just a side project.

**Build versus buy is rarely all-or-nothing.** Hermes is a packaged agent I run, not one I rewrote — I did not build the assistant, I bought it and gave it a body. But the parts that had to be exactly right for *my* trust — the data boundary, the single-writer rule, the never-act constraints — I built myself, because those are where a generic tool would not have my specific guarantees. Buy the commodity, build the part that is load-bearing for your particular risk.

**Design for trust and reversibility before features.** The most important decisions in this whole system are about what it is *not* allowed to do. An agent that acts while you are away is only as useful as it is trustworthy, and trust comes from a line you can point to — drawn, here, exactly along reversibility — not from a promise.

**Sequence value, not infrastructure.** Every wave shipped something usable; heavy infra came in only when a wave needed it. The roadmap is a series of working slices, not a long climb to a single launch.

**Review with multiple lenses before you ship.** Every wave was pulled apart by adversarial reviewers playing distinct roles before I trusted it — one playing build soundness, one playing quality and testing, and for the finance wave a third playing domain expert. This is just good product QA wearing different clothes. A single reviewer sees a single class of problem. A build reviewer, an adversarial tester, and a domain expert looking at the same thing find different holes, because they are asking different questions. And the rule underneath all of it: validate every finding before you fix it. The reviewers were sometimes wrong, and changing working code to chase a problem that does not exist is its own kind of bug.

The vision was never a clever assistant. It was a brain I already trusted, finally given a body that could act while I was away from the desk — and held to a line it cannot cross. Most of the work was drawing that line, on purpose, before building anything that needed it.

---

## Related

- [Hermes, Wave 1: Giving My Vault an Always-On Body](hermes-the-foundation.md) — the foundation post that builds the data boundary, single-writer rule, and watchdogs this overview describes at a higher level
- [Hermes, Wave 3: A Machine That Drafts From My Notes But Cannot Post](hermes-wave-3-the-machine-that-drafts.md) — the deep dive on the LinkedIn drafter and the never-post constraint summarised here
- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — the reliability-and-consolidation wave, and where the validate-before-you-fix discipline earned its keep
- [Three Claude Tools, One Vault: The Architecture Behind the System](three-claude-tools-one-vault.md) — the brain-and-hands architecture that the three-layer model in this post extends with an always-on body

---

*This post was distilled from a working session mapping the Hermes system end to end. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES
- three-layer model (vault brain / Claude Code hands / Hermes body), each one job, separation of concerns — index.md §1; CLAUDE.md "Mental model"
- problem framing: Mac sleeps so scheduled automation silently dies; always-on Linux server reliable by design — index.md §1 "The problem it solves"
- vault is live mirror; single-writer-per-file invariant (Syncthing never merges) — index.md §1, §5, glossary
- VPS ~$6/mo (Hetzner CAX11 $6.09/mo) — index.md §2 server row
- OpenRouter routing, never Anthropic direct; Anthropic-direct for Claude Code — index.md §1
- NEVER: publish/send/trade/move money/change SIP without approval; observes/drafts/alerts — index.md §1, §14; capabilities doc §1
- no posting capability in content drafter (absent not disabled); executable guard test goes red if wired in — index.md §12 Wave 3; wave-3 blog post
- reversibility framing — PM lens (intuition); maps to index.md §14 never-automate = irreversible actions
- deny-by-default .stignore; finance/health/personal never sync; new sensitive folder excluded automatically — index.md §3.1
- paid no-log/ZDR route for vault-content calls; free route can become training data — index.md §3.2; wave-3 blog
- waves value-first not infra-first; heavy infra pulled in only when a wave needs it — Hermes CLAUDE.md "Rollout — VALUE-FIRST"; index.md §12
- Wave 1 LIVE (MVP: boundary, single-writer, fail-loud harness, watchdogs, morning digest + reminder loop) — index.md §12, §2; capabilities doc
- Wave 2 LIVE (consolidation/reliability) — index.md §12
- Wave 3 LIVE (content drafter, two surfaces, raw bench) — index.md §12
- Wave 4 BUILT + reviewed (Alex+Vera+Rex) + DEPLOYED 2026-06-02 (PR #2 → main 041817c; Mac LaunchAgent + pulse live; corpus line in briefing). No soak yet — no outcomes claimed.
- morning briefing = top doable tasks + stale delegations; Wave 4 adds one corpus/progress line, no account detail crosses — capabilities doc §1, §2; index.md §8
- reminder loop: text it, parses time, ACK only after durably saved — index.md §9; capabilities doc
- investment capture: "invested ₹50k" logged not invested; rejects secret/OTP texts — capabilities doc §1 (Wave 4 BUILT)
- nightly vault maintenance (inbox, consolidation, index) — capabilities doc §1; index.md §11 Job 2
- LinkedIn drafting ~2x/week, raw bench, cannot post, human edits raw→ready — capabilities doc §1; wave-3 blog; index.md §12 Wave 3
- weekly review + alert-only crypto/work signals — capabilities doc §1; index.md §10, §11
- finance math runs Mac-side, only sanitized pulse crosses, no LLM sees finance text — index.md §3.5, §5.1; capabilities doc §2 (Wave 4 BUILT)
- packaged agent (NousResearch Hermes) run not rewritten — wave-1 blog; Hermes CLAUDE.md
- adversarial reviewers: build soundness (Alex) + quality/testing (Vera); finance wave adds domain expert (Rex); validate-before-fix — wave-1/wave-2/wave-3 blogs; capabilities doc §3 Reviews
- NO fabricated metrics/time-saved/outcomes; NO corpus rupee figure or holdings printed — per session rules; corpus figure deliberately omitted
-->

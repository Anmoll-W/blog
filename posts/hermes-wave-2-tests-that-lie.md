<!-- source_session: 2026-05-31_hermes-wave-2 -->

# Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.

*2026-06-01 · ai, systems, automation, vault · Hermes as a PA*

Wave 1 gave my vault a body — a small rented Linux server that runs all day, sends me a morning digest over Telegram, and saves reminders I text it. If you have not read that post, the short version is: my entire working life lives in a folder of Markdown notes, and Wave 1 was the unglamorous plumbing that let a server safely act on a slice of those notes while I am away from the desk.

Wave 2 is the cleanup wave. Over the year before this project, I had accumulated a sprawl of roughly eighteen small automation scripts running on my Mac — a morning briefing here, a vault tidy there, a weekly review somewhere else. Each one was useful. Together they were a mess: scattered across two machines, firing on overlapping schedules, with no single place to see whether any of them had quietly died. Wave 2 consolidates that sprawl into a handful of hardened, always-on jobs on the server. A morning pulse. A vault maintenance pass. A weekly review with a monitor watching the monitor. A job that fires reminders at the exact minute they are due.

That is the boring description, and it is true. But it is not the interesting part. The interesting part is that almost everything I am about to describe passed its tests while being completely broken — and the only reason I caught it is a review discipline I now refuse to skip. This post is about that discipline, told through the things it caught.

A note on status before I go further, because it matters and I will not blur it. Wave 2 is **built and verified, not yet live.** The code is hardened and the deploy was rehearsed against the real server in a dry run. The actual cutover and a multi-day soak happen this week. Nothing in this post is a production-outcome claim. It is a "here is what we built and how we proved it sound, before flipping the switch" report.

---

## The Discipline: Two Hostile Reviewers, On Every Single Unit

In Wave 1 I introduced a rule I will repeat here because Wave 2 is the proof of it: **a passing test suite is necessary but not sufficient.** Before any unit of work was trusted, it went through a gate on top of its own tests — two adversarial AI reviewers whose entire job is to break it. One plays build soundness: does this actually do what it claims, end to end. The other plays quality and testing: where is the failure mode you did not think of, the race, the silent fallback, the test that only looks green.

They are prompted to be hostile. And the same rule from Wave 1 still governs the loop: **validate every finding before you fix it.** A reviewer flagging something is not permission to change code — it is a prompt to go reproduce the finding against the real artifact. Real ones get fixed. Phantom ones get dropped, so I never thrash working code chasing a problem that does not exist.

Here is what that gate caught in Wave 2. Every one of these had green tests when the reviewers found it.

---

## Catch 1 — A Deploy Tool That Could Not Deploy Its Own Code

The first job of Wave 2 was a deploy tool: the thing that ships the new jobs onto the server safely, with a backup and an automatic rollback if anything looks wrong. It had a thorough test suite, and the suite was green.

Then I asked it to deploy its actual self, against the real project, and it refused. It aborted on its own repository.

Two reasons, both embarrassing in the instructive way. First, the tool runs a safety scan before every deploy — it reads the files it is about to ship and blocks if anything looks like a leaked key or password. But my test files are full of fake, deliberately key-shaped strings, because that is how you test a secret scanner. The deploy tool was trying to ship those test files, the scanner correctly recognised the fake secrets, and it blocked itself. The safety rail tripped on its own test fixtures.

Second, one of the shell scripts in the bundle was actually written in a different scripting language but carried a shell-script file extension. The syntax check ran the wrong checker against it and errored.

Neither of these was caught by the test suite for a reason that is the whole lesson: **every deploy test ran against synthetic, throwaway fixtures — freshly created fake files in a temporary folder. Not one test ever ran the tool against the real project tree.** So the suite verified that the tool worked beautifully on inputs that looked nothing like its actual job. The first time it met the real thing, it fell over.

The fix was structural. There is now one single list of exactly what gets deployed — the safety scan, the syntax check, and the deploy all read from that same list, so the scan can never check different files from the ones that ship. The syntax check now looks at each file to decide which checker to run. And there is a new, permanent rule: **every tool like this carries a test that runs it against the real project, not just against fake fixtures.** Synthetic-only testing is now treated as a known class of false confidence.

---

## Catch 2 — A Backup That Would Have Wiped the Target on Failure

This one is the catch I am most glad about, because the failure it prevented is the kind that destroys data.

The deploy tool takes a backup before it touches anything, so that if the new version misbehaves it can roll back to the previous one. Standard. The backup step was written to fail loudly — if it could not create the backup, it was supposed to stop the whole deploy cold. No backup, no deploy. Fail safe.

It did not. The reviewer traced the actual control flow and found that the failure signal was being captured inside a wrapping construct that quietly swallowed it. So if the backup ever failed, the tool would not stop. It would carry on and deploy with no backup sitting behind it. And then, if the new version failed its health check, the rollback would run against an empty backup path — which meant it would clear out the target and restore nothing. A failed backup followed by a failed deploy would not just leave the old version in place. It would erase it.

The green test suite never caught this for a precise reason: **no test ever forced the backup step to fail.** Every test let the backup succeed, so the swallowed-failure path was never walked. The bug lived entirely in the branch the tests never visited.

The fix made the failure genuinely stop the deploy, and — just as important — added a test that deliberately breaks the backup and confirms the deploy now aborts instead of proceeding. The test was then itself sabotaged on purpose to confirm it actually fails when the bug is present. A test for a safety rail is worthless if you never prove it catches the thing it guards against.

---

## Catch 3 — A Reminder Ledger That Could Split and Spam Me

One of the new jobs fires reminders at the exact minute they are due. To make sure it never fires the same reminder twice, it keeps a small ledger of what it has already fired — a "done" list it checks before sending anything.

I first placed that ledger inside the folder that syncs between the server and my Mac. It seemed natural: keep the bookkeeping next to everything else. The reviewer reproduced a nasty failure from that one choice.

The sync tool I use never merges files. If two copies of a file ever diverge, it does not reconcile them — it keeps both and leaves a conflict copy behind. So picture the ledger getting into a conflict state for any reason. The reminder job reads back what it thinks is the authoritative "already fired" list, but it is actually a stale or partial copy. A reminder it already sent is no longer in the list it can see. So it fires again. The exact duplicate-message spam the ledger existed to prevent — caused by the ledger being in a place where its own contents could not be trusted on read-back.

The fix is a rule I have now generalised across the whole system: **anything used to track "I already did this" lives on the server only, never in the synced folder.** There is a clean line now between shared content that genuinely needs to sync — your reminders, your notes — and a job's private bookkeeping, which must stay local where nothing can split it behind the job's back. A guard you cannot trust to read back correctly is worse than no guard, because it hands you false confidence while letting through the exact thing it was meant to stop.

---

## Catch 4 — One Consolidated Job, Two Morning Digests

Consolidating the scattered scripts into single jobs created a failure mode that did not exist when they were separate.

The new morning job runs several steps in sequence: send the digest, resurface any due reminders, chase stale handoffs. The harness that runs scheduled jobs retries a job if it fails. That retry is good behaviour for a single-purpose job. But once I had bundled three steps into one job, it became a spam machine: if the third step failed, the harness retried the *whole job* — which re-ran the first two steps that had already succeeded. The morning digest went out twice. The reminders fired twice.

When these were three separate scripts, one of them failing could never re-trigger the others. The consolidation I did to tidy things up is precisely what introduced the double-send. The reviewer caught it before it ever ran for real.

The fix gives each step its own small "already sent today" marker, written only after a confirmed successful send. On a retry, a step that already completed sees its marker and quietly does nothing, while the step that actually failed gets its second chance. The job is safe to retry without ever repeating a message. And — the pattern from Catch 2 again — the first version of the test for this passed even with the safety marker disabled. It was a false green. I rewrote it to drive the real send scripts and then sabotaged the marker to confirm the test genuinely fails when the protection is gone.

---

## Catch 5 — Two Tests That Passed Against Broken Code

The thread running through every catch above is this one, so I saved it for last: a green test is a claim, and claims need checking.

Twice in Wave 2, I found tests that passed against code that was deliberately broken. The way you find these is to break the code on purpose — flip a comparison, disable a guard, delete a line that should matter — and then run the test. If the test still passes, the test was never actually testing that thing. It was decoration. A test that cannot fail when the feature is broken gives you nothing but a false sense of safety, and a wall of them is worse than no tests at all, because it tells you to stop looking.

Both false-green tests were for safety-critical paths — exactly the places where a test that lies is most dangerous. Both got rewritten to drive the real behaviour, and both were re-confirmed by breaking the code and watching them go red the way they always should have.

This is now a fixed habit, not an afterthought: for anything that protects data or prevents spam, I do not trust a passing test until I have watched it fail against broken code at least once.

---

## What "Verified" Actually Means Here

I want to be exact about the bar Wave 2 cleared, because vague claims are how this kind of post becomes marketing.

The code is verified by **574 passing checks across nineteen test suites**, all green. The deploy was rehearsed against the real server in a dry run that confirmed it would ship the right files, pass every safety gate, and touch nothing it should not. Every unit went through the two-reviewer gate on its actual implementation, not just its plan. And the integration check confirmed the thing that matters most for a multi-machine setup: no two devices ever write the same file, so the sync layer has nothing to conflict over.

What has **not** happened: the cutover. Nothing in Wave 2 is running in production yet. The live switch-over and a multi-day soak — where the jobs run for real, on schedule, and I watch for any failure to surface loudly — happen this week. I am deliberately letting the Wave 1 foundation settle before I migrate anything onto it. The interesting outcomes, if there are any, belong in the next post, after the soak, not this one.

---

## The Real Lesson

If there is one thing to take from Wave 2, it is not the architecture. It is that **a green test suite told me, repeatedly and confidently, that broken code was fine.** A deploy tool that could not deploy itself. A backup that would have wiped the target. A ledger that would have spammed me. A consolidation that double-sent. Two tests that passed against code I had deliberately broken. Every one of them sat behind a wall of passing tests, and every one was caught only because something hostile went looking past the green.

The tests are not the verification. They are the first claim, and claims get checked by trying to break them. That is the whole discipline, and Wave 2 is the wave where it earned its keep five times over before a single job went live.

Next wave: the cutover, the soak, and whatever the real running system teaches me that the rehearsal could not.

---

## Update (2026-06-01): a sixth catch, and the most on-the-nose one yet

I published this, and then — the same day, working through the cutover checklist with the two reviewers — the discipline caught a sixth. I am adding it here rather than holding it, because it is the purest version of the whole point.

My wider system has a safety gate that checks every AI persona's output before it ships: banned phrases, a required status block, refusal triggers. One of its checks was quietly broken. For two of the six personas, a small pattern-matching mistake meant the "required status block" check could never match — so it always failed those two. The harmless direction of broken, as it happens: it false-*failed* rather than waving bad output through. Broken all the same.

Here is the part that belongs in this post. The bug had a passing test. And it passed because the test had been *deliberately written to match the broken behaviour* — the fixtures carried the exact malformed pattern the broken check was looking for, sitting under a comment block that explained they were shaped that way on purpose to make the broken gate pass, and instructed whoever eventually fixed the gate to update the fixtures in the same change. The test did not merely fail to catch the bug. It was built around the bug, and said so in writing.

That is the thesis of this entire post in its most literal form: a green test that was, by design, testing the wrong thing. The fix was small — correct the pattern, then de-mask the fixtures to use a real heading so they check the real behaviour, both in one change. The full suite stayed green afterward, and a direct probe confirmed the gate now passes a correct output, fails a missing one, and matches every persona's heading the way it always should have.

Still not live. The cutover and the soak are still ahead. But the green was the start of the investigation, not the end of it — once more, and in the most literal way yet.

---

## Related

- [Hermes, Wave 1: Giving My Vault an Always-On Body](hermes-the-foundation.md) — the foundation this wave consolidates onto; the "before" to this post's "after," and where the two-reviewer discipline was introduced
- [Two Models, One Pipeline: Building a Multi-LLM Peer Review Gate That Caught Its Own Silent Bug](multi-llm-peer-review-gate.md) — the same adversarial-review posture applied to a different layer, where the gate caught a bug its own green tests missed
- [When the AI Fix Is Wrong: Validating Every Finding Before You Act on It](when-the-ai-fix-is-wrong.md) — the validate-before-you-fix loop that governs every catch in this post
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the scattered Mac automation layer that Wave 2 consolidates onto the hardened server

---

*This post was distilled from a working session building and hardening the Hermes Wave 2 jobs, before their live cutover. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES (verification provenance — numbers cite live test-runner output from this session, not plan files)
- "574 passing checks across nineteen test suites, all green" — LIVE `./run-all-tests.sh` run this session: prints "ALL GREEN — 19/19 suites passed". 574 = exact sum of the per-suite green counts in that run's output: pytest 60+32+14+34 = 140; run-as-script 55+58 = 113; bash 23+25+16+27+13+29+8+3+16+41+42+50+28 = 321; 140+113+321 = 574. (Test-runner output, not a design doc. The capture-handler suite also reports 24 parametrized subtests, not counted in the 574 headline.)
- "deploy rehearsed against the real server in a dry run — ships the right files, passes every gate, touches nothing it should not" — LIVE `hermes-deploy.sh --dry-run` against the real server this session: gates printed CLEAN, all jobs listed in the plan, no test_/secret files, exit 0. No deploy occurred (dry-run only).
- "no two devices ever write the same file" — adversarial integration validation this session: zero cross-job write conflicts, zero in-place writes to Mac-owned files.
- "roughly eighteen Mac scripts → a handful of jobs" — the Wave-2 consolidation inventory (~18 Mac runners → the new server jobs).
- The five catches (deploy-can't-deploy-its-own-repo; fail-open backup → target wipe on rollback; synced idempotency ledger → re-fire; consolidated-job double-send on retry; two false-green tests caught by mutation-testing the tests) — each reproduced, fixed, and mutation-verified this session; recorded in the Hermes decisions log as outcomes, not plan intent.
- "built and verified, not yet live; cutover + soak this week" — nothing deployed to the server (dry-run only, verified); the live cutover waits for the current foundation soak to finish.
- UPDATE (2026-06-01) "a sixth catch — a persona safety gate whose required-block check used an escaped-pipe (`\|`) that the regex engine read as a literal, so two of the six personas (sage, rex) false-failed; its test fixtures were deliberately written to match the broken behaviour, under a comment block instructing not to fix the gate without updating them" — fixed this session: `\|`→`|` in `~/.claude/vault-runners/persona-gate.sh`; fixtures de-masked in `~/hermes-ops/jobs/test_job3_weekly_review.sh` + `step-refusal-regression.sh` (commit `bb64ce0`). Full suite re-run = 19/19 suites green; functional probe = single-heading PASS, no-heading FAIL, non-first-alternative (INVESTMENT STATUS) PASS. "two of the six personas" = roster of 6, sage + rex are the two with multi-heading required blocks. Still not live (cutover ahead).
-->

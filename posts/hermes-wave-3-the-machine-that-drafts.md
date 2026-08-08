<!-- source_session: 2026-06-01_hermes-wave-3 -->

# Hermes, Wave 3: A Machine That Drafts From My Notes But Cannot Post

*2026-06-01 · ai, systems, automation, vault · Hermes as a PA*

Wave 1 gave my vault a body — a small rented Linux server that runs all day, sends me a morning digest, and saves reminders I text it. Wave 2 moved my scattered automation onto that hardened server and taught me, five times over, that a green test suite will confidently tell you broken code is fine. Wave 3 is where the body starts to write.

The new capability is narrow on purpose. The always-on server now reads the signals already sitting in my vault — the ideas I capture in daily notes, the "next week" suggestions from a weekly analyst report, my published blog posts, my decision log — and from them it drafts LinkedIn posts. It drops each one on a bench as a raw candidate, for two surfaces: my personal account and a brand page. That is the whole feature. It keeps the blank page stocked.

There is one hard line through the middle of it, and the rest of this post is really about that line. **Nothing the machine produces ever posts itself.** It drafts. A human edits the draft from raw to ready and posts it by hand. Always.

That sounds like a small caveat. It is the entire safety model. Because the moment you build a system that ingests your own notes and writes from them, you have built a prompt-injection surface — and this time the adversarial review did not catch a swallowed error or a false-green test. It caught my notes attacking the machine that reads them.

---

## What "Cannot Post" Actually Means

I want to be precise about the invariant before anything else, because a vague promise of "a human stays in the loop" is exactly how these systems quietly drift into posting on their own.

The machine cannot post because there is no posting capability in the code at all. It can read sources, it can write a draft file, it can send me a Telegram nudge that says new drafts are waiting. It cannot reach LinkedIn. Every draft it writes is hardcoded to a single status — raw — which my downstream pipeline only ever lists for review, never acts on. And only one writer ever creates these files: the server creates them once and never edits them again. I do all the editing on the Mac.

This is not a convention I am trusting myself to honour. It is an executable guard — a test whose entire job is to assert that nothing in the drafting path can post, that drafts are always written as raw, and that the writer stays single. If someone (including a future, lazier version of me) ever wires in a posting call, that test goes red. The human gate is not a habit. It is enforced.

---

## The Privacy Half of the Line

There is a second invariant that matters just as much, and it is about where my notes go when the machine thinks about them.

Every drafting call carries vault content — that is the point of the feature. So every one of those calls is pinned to a paid, no-log route: a model endpoint that is told, in the request itself, not to retain or train on the prompt. The relevant control is set to deny, explicitly, on every call. A free route is not merely discouraged here; a route guard makes it unreachable for any prompt that carries vault content. External-only work — the kind that touches no private notes — may use a cheaper route. Anything carrying my brain may not.

The reason for the strictness is unglamorous and worth saying plainly: a free model route can mean your prompt becomes training data. When the prompt is a slice of your own notes, "free" is the most expensive option there is. So the guard inspects the actual outgoing request before it leaves, confirms the no-log control is set, confirms the route is a paid one, and aborts if either is wrong. Privacy is not a setting I remembered to flip once. It is checked on every call, by code that fails the call rather than risk a leak.

---

## The Interesting Part: My Own Notes as an Attack

Here is the tension that makes Wave 3 worth a post rather than a changelog line. A system that reads your notes and feeds them to a model is, structurally, a prompt-injection target — and the thing doing the injecting is your own vault.

The defence is straightforward in principle. When the machine sends a note's content to the model, it wraps that content in a marker — a tag that says, in effect, "everything between here and the closing tag is data to write about, not instructions to follow." The model is told to treat the wrapped region as quoted material. This is the standard move for feeding untrusted text to a model, and I had it in place.

The two-reviewer adversarial gate — the same hostile pair from Wave 1 and Wave 2, one playing build soundness, one playing quality and testing — went looking for ways through it. They found three real holes across two rounds of review, and they found them before any of this shipped. Every one of these had passing tests when it was caught.

**A note could break out of its own wrapper.** The wrapper works only if the wrapped content cannot contain the closing tag. But a note is just text I wrote, and nothing stopped me from writing the literal closing tag inside a note. If I had — by accident or by copying something in — the wrapper would close early, and everything after it would land in the model's instruction space instead of its data space. My own note would stop being quoted material and start being a command. The reviewer reproduced exactly this. The fix neutralises the closing tag inside any content before it is wrapped, so a note can never end its own quotation.

**A crafted identifier could forge a finished status.** Each draft carries a small block of metadata at the top, and one field in it is the status — the raw flag that keeps the human gate intact. The reviewer found that a carefully crafted source identifier could inject extra lines into that metadata block — including a forged status of ready. Read that again, because it is the whole nightmare of this wave in one bug: a value derived from a source could promote a draft past the human gate, making the machine's output look like something a human had already approved. The exact invariant the entire system exists to protect, defeated by a string. It was caught, reproduced, and fixed — the metadata is now constructed so that source-derived values cannot inject structure, and the status cannot be anything but raw.

**Collisions could silently drop content.** The machine keeps a ledger of what it has already drafted, so the same capture or blog post never gets drafted twice. That ledger keys on a short fingerprint of each source item. The reviewer found cases where two genuinely different items could land on the same fingerprint — which would make the second one look already-done and vanish without a trace. Not a crash. Not an error. Just content quietly never drafted. The fix made the keys collision-proof, and there is now a regression test for every one of these.

None of the three threw an exception. None of them failed a test. They were caught because something hostile sat down and tried to weaponise the input, which is the only way these are ever caught.

---

## And Then a Second Review Found Two More

The injection holes came from the adversarial gate on each unit. A separate end-to-end pass — looking at the whole flow rather than each piece — caught two more before go-live, and both are the unglamorous kind that bite you in production.

The first was a duplicate-draft-on-retry bug. The drafting job, like every job on this server, retries if it fails. But if it had already written a draft and then stumbled on a later step, the retry would write the draft again — the same spam-on-retry failure mode I hit in Wave 2 with reminders, in a new costume. The fix makes the draft-and-record step safe to repeat: a draft already written is recognised and not duplicated.

The second was a privacy hole hiding in a configuration rule. To bring two of the new sources onto the server, I had to widen the slice of the vault that syncs to it — and the rule I first wrote was too broad. As written, it would have copied private session notes to the server along with the intended sources. This is precisely the boundary Wave 1 was built to protect, and I had quietly cracked it open while adding a feature. The review caught it before the rule went live. The corrected rule names exactly the two paths that should cross and nothing else.

---

## The Same Lesson, Sharper

If Wave 2's lesson was "the tests passed, the code was broken anyway," Wave 3 is the sharpened version of it.

The tests were green here too. They did not catch a note breaking out of its wrapper, a string forging an approved status, a fingerprint collision dropping content, a retry double-drafting, or a sync rule leaking private notes. Not one of those is the kind of bug a test suite written by the person who wrote the feature tends to imagine. They were all found by deliberately pointing something hostile at the system and asking how it breaks.

So the safety of a machine that writes from your own notes does not come from green tests. It comes from three things stacked together: a human-approve gate that the code cannot bypass, privacy-pinned routing that fails the call rather than leak, and a reviewer actively trying to break both. Remove any one of the three and the green tests will still be green while the system is quietly unsafe.

---

## Where It Stands

I will be exact about status, because vague claims are how this kind of post turns into marketing.

Wave 3 is built, reviewed, deployed, and live on both surfaces. The bench now self-stocks roughly twice a week from the four vault sources. The drafts land as raw candidates, attributed to the right surface, and I edit and post them by hand exactly as I did before — the difference is that the blank page is no longer my problem to solve cold.

What I am deliberately not claiming: any result. It just went live. There is no soak behind it yet, no engagement number, no before-and-after on my writing throughput. Those belong in a later note, if they turn out to be true, after the system has run for real long enough to have something honest to say. For now the claim is only this: the machine drafts from my notes, on a schedule, through a route that cannot leak them — and it cannot, by construction, post a single word without me.

That was always the point. Not a clever ghostwriter. A body for a brain I already trust, given exactly enough rope to keep the page stocked, and no more.

---

## Related

- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — the wave that established this post's lesson; Wave 3 is the sharper version, where green tests missed prompt-injection holes the adversarial review caught
- [Hermes, Wave 1: Giving My Vault an Always-On Body](hermes-the-foundation.md) — the foundation and the privacy boundary this wave both builds on and nearly cracked open while adding sources
- [Hermes, Wave 4: Finance Intel From a Server That Cannot See My Finances](hermes-wave-4-finance-intel-off-server.md) — the next wave, where the same "enforced in code, not good intentions" gate that kept this drafter from posting is applied to money: the server can never trade or move a rupee

---

*This post was distilled from a working session building, reviewing, and deploying the Hermes Wave 3 content drafter. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES
- "drafts LinkedIn posts from 4 vault signals (daily-note captures, analyst 'next week' ideas, published blog posts, decision log), drops on bench as status: raw, two surfaces personal + brand page" — wave3-content-engine-design.md (design); session log MW3-1 content_selector reads 4 sources; UPDATE 2 live on both surfaces
- "nothing auto-posts; human edits raw to ready and posts manually; enforced by executable guard test (no posting capability, hardcoded status: raw, single-writer)" — design Invariants 1+3; session MW3-4 test_no_autopost.sh (8 tests, commit 2385ab6)
- "every drafting call carries vault content uses paid no-log/ZDR route; request pins data_collection=deny; route guard makes free route unreachable for vault content" — design Invariant 2 + Decisions; session MW3-0 route_guard.py (commit 310d5c1, +4c1c026): blocks :free on vault prompt, requires provider.data_collection==deny, paid-model allowlist, external-only may use free
- INJECTION HOLES caught by two-reviewer gate-3, each reproduced+fixed+regression-tested:
  (a) note containing literal closing tag </external_content> breaks out of wrapper into instruction space — session UPDATE "drafter prompt-injection escape (</external_content>)" BLOCKER
  (b) crafted identifier injects YAML frontmatter, forges status: ready, sneaks past human gate — session UPDATE "YAML-frontmatter injection (forged status: ready)" BLOCKER
  (c) hash/fingerprint collisions silently drop content — session MW3-1 "unescaped | to NUL separator" + "undated analyst report, then empty-date cross-file collision, then key on filename stem" BLOCKERs
- SEPARATE end-to-end review caught: duplicate-draft-on-retry (session next-steps MW3-2 crash-window note: write, then fsync, then append atomic ledger; "duplicate-draft" idempotency) + over-broad sync rule copying private session notes (session "slice-widen too broad", corrected to name exactly Knowledge/decisions.md + Projects/PM-Code/Posts/Drafts/ only; staged then applied with validate.sh 63/63)
- "same two hostile reviewers, build soundness + quality/testing" — Wave 1 + Wave 2 posts (Vera + Alex); session gate-3 references
- "built, reviewed, deployed, live both surfaces; bench self-stocks ~2x/week" — session UPDATE 2: hermes-deploy.sh deployed 5 files, cron 0 22 * * 3 + 0 14 * * 0 (Wed+Sun), both surfaces live-verified via real ZDR calls, drafts status: raw synced back to Mac
- NO soak result, NO engagement numbers claimed — session UPDATE 2 "Wave-3 is autonomous now"; MW3-5 closure/soak still ahead
-->

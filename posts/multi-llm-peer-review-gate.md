<!-- source_session: 2026-05-10_multi-llm-auth-image-gen -->

# Two Models, One Pipeline: Building a Multi-LLM Peer Review Gate That Caught Its Own Silent Bug

*2026-05-10 · ai-agents, claude-code, gemini, multi-llm, eval*

I wired a second model into my Claude pipeline. Claude proposes, Gemini reviews, the gate fails closed if either side disagrees. Within an hour of the system going live, the verify pass caught a silent bug in the system itself. This is the build, why I did it, what it does, and what the verify pass actually found.

## What I built

A four-mode peer review pipeline that runs on every vault file flipped to `status: ready` — content posts, code changes, design decisions, research notes. The wrapper invokes Gemini through the official CLI in headless mode, feeds it the file plus a mode-specific prompt, and validates that the response contains the required H2 sections for that mode (`## Verdict`, `## Risks`, `## Suggestions`, `## Disagreement signal`, plus a `## Counter-thesis` section for decision and research). If the sections are present and the verdict is not red, the gate passes. If anything is missing or the verdict is red, the file does not progress.

A second runner generates images for LinkedIn drafts that include a `## Gemini Image Prompt` section. Same env file, same auth, different model. The dispatcher script fans out: image generation fires async via `nohup`, the content review runs synchronously and writes its evaluation inline. A launch agent watches the vault and triggers the dispatcher whenever a file mtime changes inside `Posts/` or `blog/` or `Projects/`. Recursion stops are explicit: anything inside the runner's own directory, the persona files, or the skill files do not trigger the dispatcher onto themselves.

End to end: twelve scripts, one launch agent, one env file at mode 600, one state file that tracks last-fired mtimes per path. Cold-start guard prevents the dispatcher from firing onto every existing file the first time launchd loads the plist. Single-instance lock prevents concurrent runs.

## Why a second model

Two reasons, neither of which is redundancy.

The first is that two models trained on different data and aligned by different teams disagree about different things. Claude over-indexes toward agreement when the user has stated a preference; Gemini over-indexes toward enumerating every alternative whether or not the user wants them. Where they agree, the recommendation is robust. Where they disagree, the disagreement itself is the signal: one of the two has spotted something the other missed, and the file should not progress until I have read both reviews and decided which is right. A single-model gate cannot produce this signal. Two models can.

The second is posture. Claude is the proposer in this system. It writes blog posts, drafts LinkedIn copy, generates code, designs schemas. Asking the same model to evaluate its own output is the AI-coding equivalent of asking a developer to write the test suite for the code they just shipped — they tend to test what they remember writing, not what is actually there. Gemini is the adversary. It does not know what the previous run produced; it sees only the artifact and the prompt that names the failure modes to look for. The asymmetry is the point.

## What it does

Four review modes, each with a different prompt and a different validation contract.

**Code review.** The mode I tested first against a small banking-style fixture: a money-transfer handler with a SQL injection on the user-provided account number, a non-atomic transfer that could double-spend under concurrency, a missing authorization check, and a `fetchone()` whose return value the caller dereferenced without checking for `None`. Gemini flagged all four with line numbers and a concrete suggested fix for each. Not boilerplate. Not generic. Specific to the fixture.

**Content review.** Used on this very post and on the LinkedIn draft for it. Catches the failure modes that show up in writing: weak hooks, fabricated time durations, scale claims without citations, displacive summaries that paraphrase the source too closely. Pairs with my eval persona's existing rubric — the persona handles voice and rubric scoring, Gemini handles independent challenge.

**Decision review.** Adds `## Counter-thesis` to the required sections. Forces the second model to argue the opposite of the conclusion before it agrees. The discipline catches the case where the proposer collapsed onto a default without considering the alternatives.

**Research review.** Adds `## Gaps` and `## Sources to verify`. The mode that runs against analyst reports, project research notes, market scans. Looks for unverified claims and missing context, not stylistic issues.

The image generator is wired up but parked. The Generative Language API free tier has an image-model RPD limit of zero across every available image model. Live calls return HTTP 429 with `limit: 0`. The runner detects the 429, logs `SKIPPED api-error http=429`, and exits zero. Fail-open is the design: when billing is not enabled, the gate degrades quietly. The day billing is enabled, the same script ships images without a code change.

## How it helps

Three concrete things, each measurable against the system before it existed.

One. The gate catches drift between what I asked for and what the proposer produced, before the artifact moves to a published or shipped state. Specifically: a LinkedIn draft with an unverified scale claim does not pass content review, because the second model has no investment in defending the proposer's word choice. A schema migration with a missing index does not pass code review, because the second model is reading the diff cold without the context that made the omission feel reasonable.

Two. The dispatcher decouples review from authorship. The proposer ships into the vault. The watcher fires asynchronously. The reviewer runs without me sitting in the loop. Reviews land in the file before I open it, and the verdict is the first thing I see when I do. This is the difference between an eval pass that sometimes runs and a gate that always runs.

Three. The cost is bounded by free-tier policy, not by my discipline. Headless calls hit the API key on a project with no billing account linked. Interactive calls hit the OAuth track on the Google AI Pro consumer plan. Either path runs into the free-tier limit before it bills, and the runner is built to fail open on rate limits rather than fail loud. The cost can never escape the policy because the policy is enforced upstream.

## The verify

I asked my eval persona to audit the system end to end. Six scenarios across the four review modes, plus the image generator, plus the dispatcher fan-out, plus the env-file auth, plus the recursion stops. The persona ran the wrapper script against the banking fixture and watched what came back.

The script returned. Exit zero. The persona moved on.

Then I went back to look at the output file. It was empty.

Here is the sequence the eval persona reconstructed.

1. Wrapper sources the env file. The API key is loaded.
2. Wrapper invokes the Gemini CLI with a prompt on stdin.
3. The CLI loads the workspace — the wrapper's working directory. It checks whether the workspace is in its trusted-directories list. It is not.
4. The CLI writes `Gemini CLI is not running in a trusted directory` to stderr.
5. The CLI exits zero.
6. The wrapper sees `$?` is zero. It assumes the call succeeded. It reads stdout. Empty string. It logs a warning that some required H2 sections are missing and continues.
7. The wrapper writes the empty review to its destination and exits zero.

Every layer above the CLI saw a successful run. The dispatcher logged green. The launch agent logged green. The post passed every gate it was supposed to fail.

A wrapper with `set -eu` does not catch a successful pipe that produced nothing. A test against a fixture does not run when the live invocation reports success. The launch agent retries on failure but does not retry on a successful run that produced nothing. This is the bug class — exit zero, empty stdout, warning to stderr that nobody reads — that hides inside any pipeline where a child process can decline a job without raising. The CLI made an explicit and reasonable refusal. The exit code was the wrong place for it.

## The fix

Two lines.

```
export GEMINI_CLI_TRUST_WORKSPACE=true
```

Once in the env file the wrapper sources. Once inline at the top of the wrapper itself, with `${VAR:-true}`, in case the env file source ever fails or the script is invoked standalone. Belt-and-braces because the cost of being wrong is empty review files written to vault paths that drive automation downstream.

Re-ran the same fixture after the fix. The CLI returned a structured review: SQL injection flagged with the line number, the race condition called out, the missing authorization check named, the dereference bug surfaced. Standard out had content. The wrapper validated all four required H2 sections. The destination file was correct.

## What this exposed

Two things, neither novel, both repeatedly under-respected.

One. Wrapper scripts should validate the output shape, not the exit code. `set -euo pipefail` catches a failed pipe component. It does not catch a successful pipe that produced nothing usable. The wrapper now treats empty stdout from the model as a hard failure — same path as auth-skip, same fail-open stub written, exit zero only after the stub is in place. The downstream gate sees real content or a tagged skip; it never sees silence.

Two. Eval gates need to live above the wrappers, not inside them. The reason I caught this at all is that I had a separate eval pass auditing the system end to end. The wrapper itself reported green every time. The eval pass read the output files, saw they were empty, and only then traced backward to the CLI's stderr. Without that pass, the bug ships and runs every day under launchd, and every post that should have triggered a content review goes through with empty results that nobody notices because the logs all say PASS.

The general form: a tool that exits zero on refusal is a tool that needs an output-shape check from above. The exit code reports compliance with the call. The output reports compliance with the work.

## What I learned

A multi-LLM review system is supposed to catch the things one model misses. The first thing it caught was the wiring of the system that runs it. That is the only outcome a verify pass should produce — either a pass with evidence, or a found bug. A green report with no evidence found nothing because nothing ran.

The pipeline is now four review modes and one image generator across two auth tracks. The number of moving parts that can each independently exit zero with no work done is larger than I would have predicted before counting them. Output shape is now the contract. Exit codes are signal, not truth.

The disagreement between two models is what makes a multi-LLM pipeline worth the cost. The disagreement between the script's exit code and the script's actual output is what makes the system survive contact with launchd.

---

**[2026-05-10](../README.md#all-posts)** · [![ai-agents](https://img.shields.io/badge/ai--agents-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![claude-code](https://img.shields.io/badge/claude--code-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![gemini](https://img.shields.io/badge/gemini-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![multi-llm](https://img.shields.io/badge/multi--llm-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts) [![eval](https://img.shields.io/badge/eval-6f42c1?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](hermes-wave-2-tests-that-lie.md) — the same two-reviewer gate caught five defects a green test suite missed, including two tests that passed against deliberately-broken code
- [The Eval Layer Caught Me Violating My Own Rules](the-eval-layer-caught-me.md) — the eval persona that ran this audit and the posture that catches its own drift
- [The Eval Agent: Adding a Quality Gate to an AI Workflow](the-eval-agent.md) — the original install of the eval persona this verify pass came from
- [AI Runners That Remember](ai-runners-that-remember.md) — the runner architecture this multi-LLM gate plugs into
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md) — the same silent-failure shape one layer down: a scheduler that reports success while the underlying exec silently refuses
- [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](claude-appends-text-after-json.md) — another model-pipeline silent failure where the call succeeds and the parsed result is wrong
- [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](claude-code-boot-sequence-as-infrastructure.md) — the same posture applied one layer higher: shape-check the boot, not the launch
- [We Designed a Multi-Model Router. Then We Asked One Question.](right-model-wrong-problem.md) — how this peer review gate's adversarial Gemini pass was used to evaluate a full routing architecture, then overturned by a 15-minute CLI test
- [Hermes, Wave 1: Giving My Vault an Always-On Body](hermes-the-foundation.md) — the same adversarial-review posture across fourteen micro-waves, including a validate-before-fix catch on a crashing date parser
- [Your Agents Are Only as Good as the Context You Program Them With](context-is-the-program.md) — why a reviewer with a different objective is structurally different from a second pass of the same model: the adversarial agent caught that the cleanup was incomplete precisely because it wasn't trying to confirm the work was done

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- source_session: 2026-05-11_gemini-flash-research-integration -->

# We Designed a Multi-Model Router. Then We Asked One Question.

*2026-05-11 · ai-agents, multi-model, decision-frameworks, claude-code, gemini · Vault as OS*

The architecture was clean. A central model registry. A Groq classifier that reads every incoming task and routes it to the right model in under a second. Claude Opus for creative work and memes. Claude Haiku for code. Gemini Flash for web-grounded research. Gemini Pro as the adversarial reviewer. All free. All wired.

We had the capability map. We had the routing table. We had a test that proved the Claude CLI `--model` flag works on a Max subscription. We were ready to build.

Then someone asked: "But do you even need this?"

---

## What we were trying to solve

The premise was right. Different models genuinely are better at different things. Claude Opus produces the most nuanced creative writing: the voice that makes memes land and LinkedIn hooks stop the scroll. Claude Haiku is Anthropic's fastest coding model, built for agentic and computer-use tasks. Gemini Flash has a 1-million-token context window and live web search built in. These are real differences.

The gap in our setup: everything routes to Claude Sonnet by default. No task typing. No specialisation. Just the same model for memes, for debugging, for research, for planning.

A smart routing layer would fix that.

---

## The eval that almost blocked it

Before building, we ran Vera's decision eval: six stress-test questions from a rubric, plus a Gemini adversarial pass in `decision` mode. Gemini returned **BLOCK** with this as its primary concern:

> Claude Max is a subscription. The vault runners use `claude -p` (headless CLI). You cannot currently route `claude -p` calls to specific sub-models without either changing settings.json each time (fragile) or using the paid Anthropic API (contradicts the "free" constraint).

This was a legitimate blocker. If CLI model selection doesn't work on Max, the entire routing architecture collapses. You'd need paid API tokens to route between Claude models, which defeats the point on a flat subscription.

We ran the test:

```bash
claude --model claude-haiku-4-5-20251001 -p "Reply with only: HAIKU_OK"
claude --model claude-sonnet-4-6 -p "Reply with only: SONNET_OK"
claude --model claude-opus-4-7 -p "Reply with only: OPUS_OK"
```

All three responded immediately. On Max. No extra cost.

Gemini's BLOCK became FIX-FIRST. The critical assumption was false. We could build.

---

## The question that changed everything

We had green lights. Test passed. Architecture validated. Then Vera surfaced the real blind-spot question:

**"Are you actually routing wrong today?"**

Before building a routing layer, look at the last 30 vault runner logs. Which tasks produced low-quality output? Which ones hit quota limits? If the answer is "none that I noticed," you're building a solution to a hypothetical.

We checked. No evidence of output quality failures from wrong-model routing. No quota pressure. No tasks where Sonnet was clearly the wrong choice and the output suffered for it.

The full routing system was a solution without a problem.

---

## What actually shipped

Two surgical changes. One new file. No registry. No Groq classifier. No global dispatcher.

**`gemini-research.sh`**: an 80-line helper that calls Gemini Flash via the REST API with `google_search` grounding enabled. Tries live web search first; falls back to training knowledge on quota limits. Always exits 0, never blocks the caller.

**`run-morning-briefing.sh`**: now calls `gemini-research.sh` before the Claude invocation with a world-pulse query (LW/SEO developments, AI developer tools, PM practices). If Gemini returns something, it's injected as `RESEARCH CONTEXT` into the Claude prompt. Claude adds an optional `**World pulse:**` line to the brief, only if directly relevant to active projects.

**`run-weekly-review.sh`**: same pattern, different query: trending LinkedIn topics for product managers and AI builders this week. Claude uses this to ground the "Next week content plan" section in what people are actually discussing, not just vault captures.

If Gemini Flash fails or returns empty, both runners proceed exactly as before. Zero fragility introduced.

---

## The patterns worth keeping

**Test the blocker before designing around it.** Gemini's concern about CLI model selection was valid. But it took 15 minutes to disprove. We could have spent hours redesigning the architecture to work without per-call model selection, and the problem didn't exist.

**"Do I even need this?" is the most useful question before building.** The routing architecture was genuinely good. The capability map was accurate. The model assignments were research-backed. None of that matters if the problem it solves is hypothetical. The time to ask the question is before you write the first file, not after.

**The right scope is the one that fills a real capability gap.** Gemini Flash adds something Claude Max cannot do natively: live web-grounded research. That's a real gap. Routing Haiku vs Sonnet on a flat-fee subscription? The quality delta is real, but there's no evidence it's causing harm today. Build for the gap, not for the theory.

---

The full routing system will probably get built eventually, when there's evidence of quota pressure, or when ChalotripBot has real users burning API tokens daily. The architecture is ready. The test confirmed it works.

For now: 80 lines, two runners, one real capability added.

That's the right amount of system for the problem that actually exists.

## Related

- [Two Models, One Pipeline: Building a Multi-LLM Peer Review Gate That Caught Its Own Silent Bug](multi-llm-peer-review-gate.md): the Gemini review gate that this session used to evaluate the routing decision; the infrastructure that ran the adversarial pass
- [I Cherry-Picked a Viral Cost-Cut Post](cherry-picking-the-cost-post.md): the same cherry-pick discipline applied here: evaluate each claim independently, adopt what has evidence, reject what is marketing
- [AI Runners That Remember](ai-runners-that-remember.md): the runner architecture that the Gemini Flash research leg plugs into; how morning-briefing and weekly-review carry memory across runs
- [The Claims Were There. The Sources Were Not.](claims-without-sources.md): the Vera-catches-a-wrong-default pattern again: here it caught a provenance tagging rule that would have drifted toward false authority
- [The Eval Layer Caught Me Violating My Own Rules](the-eval-layer-caught-me.md): Vera's eval pattern in action: the same rubric-plus-adversarial-pass process used to stress-test this routing decision
- [AI Credits Are Infrastructure. Start Treating Them That Way.](ai-credits-are-infrastructure.md): what came after the routing decision: a full cost audit that validated model routing as the single biggest lever, and the PM framework for reading the bill
- [My AI Agents Got Dumber. It Was Not a Model Downgrade.](why-my-agents-got-dumber.md): the course correction to this router: the right model is a per-task judgment, and a blanket "use the cheap model" rule was quietly making every dispatched agent dumber than the session that called it
- [I Ran the Stats Hoping to Prove It Worked. It Did Not, and That Was the Deliverable.](null-result-was-the-deliverable.md): the same discipline applied to a statistical result instead of a routing decision. A finding that only holds under one framing is not a finding

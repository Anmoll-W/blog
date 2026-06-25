<!-- source_session: 2026-06-25_auditing-another-pms-agent-repo -->

# What I Learned Auditing an AI Agent Repository Built by Another Product Manager

*2026-06-25 · ai-coding, code-review · Reading Other People's Agents*

> Reading someone else's agent code is the fastest way to see your own blind spots. You recognize the decisions you also make, and you finally see the ones you make without thinking.

## Context

I spent a session reading through an AI agent repository built by another product manager. Not an engineer who happens to build product. A product manager who, like me, reached for Claude Code and assembled something that works. The repository ran. It did real work.

I did not go in to grade it. I went in to learn. When you read code written by someone who shares your background and your constraints, you are really reading a mirror. The decisions that felt obvious to them are the decisions you also make on autopilot. This post is what I took away. It is a product perspective, not an engineering teardown.

## The Structure Tells You What the Author Was Afraid Of

The first thing I read was not the logic. It was the shape of the repository. Where the files live, what is named carefully, and what is named in a hurry tells you what the author was thinking about and what they were avoiding.

This repository had a beautifully organized prompt layer and a chaotic error-handling layer. That is not an accident. The author spent their attention where they felt confident and skipped where they felt uncertain. Prompts are the part a product manager understands intuitively, because prompts are specification writing in a new costume. Error handling feels like engineering, so it got deferred.

I do the same thing. I had just never seen it laid out so clearly until I was looking at someone else doing it. The takeaway is uncomfortable and useful: the polished part of your codebase is where you were comfortable, and the comfortable part is rarely where the risk lives.

## The Agent Did Too Much in One Call

The central agent loop tried to do everything in a single model call. Parse the request, decide the action, format the output, and handle the edge cases, all in one prompt. It worked in the demo. It would not survive contact with real users at scale, and it would be expensive every time it ran.

This is the most common pattern I see in agent code written by people who learned the language model first and traditional code second. The language model becomes the hammer for every problem. The model is genuinely good at most of these steps. But good is not the same as cheap, and good is not the same as reliable.

Here is the framework I pulled out of this, the one question I now ask of every agent step:

> Does this step need judgment, or does it need a decision tree I could draw on paper?

If I can draw it on paper, it does not belong in a model call. It belongs in ordinary code that runs the same way every time and costs nothing after it is written. The model should be reserved for the steps that genuinely require interpretation: ambiguity, language, summarization, taste. Everything else is plumbing, and plumbing should be deterministic.

The repository I read used the model for several steps that were pure plumbing. Pulling those out would make it faster, cheaper, and more predictable, without losing anything a user would notice.

## The Tests Were Testing the Wrong Layer

There were tests. That is already ahead of most agent repositories built by non-engineers. But the tests checked that functions returned a value, not that the agent made the right decision. They confirmed the machine was running. They did not confirm it was running correctly.

This is the gap that bites product managers specifically. We are trained to care about outcomes, so it is strange that our test code so often checks mechanics instead of outcomes. A test that confirms the settlement math is correct is worth more than ten tests that confirm a function did not throw an error. The author had written the second kind and skipped the first.

The lesson I am taking back to my own work: write the test that would catch the failure you would be embarrassed to ship. Not the failure that is easy to write a test for.

## What This Changed About How I Read My Own Code

The most valuable part of the session was not anything I found in the other repository. It was what I noticed about my own habits once I had names for these patterns. I polish the parts I understand. I reach for the model when ordinary code would do. I test that things run instead of testing that they are right. I knew none of this clearly until I watched another product manager do the same three things and felt the recognition.

Reading another person's agent code is cheap, fast, and more honest than reading your own. You cannot rationalize someone else's shortcuts. You can only see them. And once you have seen them in their work, you cannot unsee them in yours.

If you build with AI tools, find a repository written by someone with your background and read it end to end. Not to judge it. To find yourself in it.

## Related

- [The Write-Only Trap](the-write-only-trap.md) — what I built in response to the top finding from this audit: the observation log was capturing data that nothing consumed; this post covers the three-piece system that closed the loop
- [When the AI Fix is Wrong: What Senior Review Catches That Pattern Matching Misses](when-the-ai-fix-is-wrong.md) — what surface-level pattern matching misses when it does not trace the full data path.
- [How to Audit a Production Codebase Against Its Own Support Data](auditing-plugin-against-support-data.md) — a method for reading an unfamiliar codebase by following the evidence rather than the architecture.

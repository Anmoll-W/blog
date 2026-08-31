<!-- source_session: 2026-08-31_thepmcode-skills-hub -->

# Seven Skills That Have to Show Their Work

*2026-08-31 · claude-code, skills, ai-agents, product-management*

You are in standup. Your engineer says the webhook processing is causing race conditions and you need a queue. You nod. You write it down. That afternoon your manager asks what the sprint is about, and you hear yourself say "infrastructure work," and you cannot say more.

I have had that exact moment. Most product managers have. So I built a small tool for it, then six more for the moments like it, and this week I put all seven in one place: [thepmcode-skills](https://github.com/Anmoll-W/thepmcode-skills), open source, for Claude Code.

This post is not about the folder. It is about what each one does, and who it helps.

## The one rule they share

Before the list, the thing that ties them together, because it is the reason they exist.

Every skill names the mode it is running before it says anything else, sources every claim it makes, and tells you plainly when it checked nothing. A confident guess and a researched answer read identically on the screen. The only fix is to force the tool to say which one you are holding, every single time. That rule is what separates a tool you can repeat in a room where you will be challenged from a tool that sounds good and gets you caught.

## Understand what was just said

**Decoder** takes a technical term, a pasted document, or the sentence your engineer just used, and explains it at the depth the moment allows. Standup phrasing gets four sentences and no diagram. A pasted spec gets the full treatment. You never pick the depth; it reads that from how you asked.

It does more than define the thing. It hands you a character whose job makes the failure obvious, the mistake most people make, and one question you can carry back to the engineer. The last line of every answer says what it checked, or admits it checked nothing and this is general principle. Its rules are graded by an adversarial suite of thirty three fixtures, each one built to make it fail in a single specific way.

Who it helps: the PM who understood every word in the meeting and none of the meaning.

## Decide before you commit

**Decision Support** applies pressure to a decision before you are married to it. Four modes. EVALUATE judges an idea and lands on Strong, Weak, or Pivot Required, then writes the one line that would end the idea and when to look for it. GRILL interviews you a single question at a time, as the engineer you will have to answer to in three months. INVERT takes a solution someone already proposed and grades two things apart: does the pain exist, and has the fix ever been tested. CHALLENGE argues against a position you are not defending, steelman first.

It will not cheerlead. Enthusiasm is not evidence, and agreement is not a service.

Who it helps: the person whose idea has felt good for a week because nobody has pushed on it yet.

**pg-startup-eval** is the same instinct pointed at a whole business. It runs an idea through seventeen investor and founder frameworks, pulls live market research, sizes the market from the bottom up, and refuses to end on a maybe. Strong, Weak, or Pivot, with the three fatal flaws named.

Who it helps: the founder whose friends were all polite.

## Ship it without the bug

**Code Quality Suite** looks for the bug before it ships and will not tell you the code is fine without pasting the test output that says so. Five modes cover finding bugs, an OWASP Top Ten security pass, a test plan built from the code, a browser test run, and a whole-repo pre-launch audit where six specialists run in parallel and a seventh, separate agent confirms or rejects each finding. Every finding carries a file, a line, and a reproduction, or it gets labelled suspected and ranked last.

Who it helps: the team about to approve a branch nobody has actually tested.

**Design Review Suite** turns "this looks off" into a finding with a rule number on it. Five modes score the Nielsen heuristics, measure contrast and tap targets against WCAG 2.2, run a polish pass, gate a launch, or pick a palette and check its contrast before any code uses it. A subjective preference is allowed, but it is labelled TASTE and always ranks below a real accessibility failure.

Who it helps: the two people staring at a screen who both agree it feels wrong and cannot say which rule it breaks.

## Talk to the model, and keep what you learned

**Prompt Generator** turns a one-line brief into a structured prompt with role, constraints, and output contract kept separate. In the example in its own README, that took a request from three hundred eighty tokens of hedged prose down to ninety five tokens of instruction.

Who it helps: anyone whose output is nearly right and who cannot tell which instruction is wrong.

**Teach** catches the lesson a real session produced without breaking the session it came from. When the work earns a lesson, it writes six lines, logs the concept, and schedules it to come back tomorrow. Ask for a review and it quizzes you on something you actually built, never a definition. Before it quizzes you on a card that names a file or a system, it checks that the thing still exists, and deletes the card if you tore that system out.

Who it helps: the person who learns by shipping and keeps re-learning the same thing because nothing ever held it in place.

## Install one, or all seven

```bash
git clone https://github.com/Anmoll-W/thepmcode-skills
cp -r thepmcode-skills/skills/decoder ~/.claude/skills/decoder
```

Swap `decoder` for any skill name to install a different one. Each activates on its own inside Claude Code when the moment it was built for shows up.

I did not set out to build seven skills. Each one started as a workflow that broke in the same place every week, got written down as a contract, then got a check bolted to the bottom so it could not quietly drift. The hub is just where they live now.

## Related

- [A Real Problem Is Not a Reason to Build](a-real-problem-is-not-a-reason-to-build.md): the INVERT mode inside Decision Support, and the day it told me to delete my own tool instead of moving it.
- [The Eval Was Grading My Config, Not My Skill](the-eval-was-grading-my-config.md): how the Decoder eval suite in this hub gets measured, and the noise that nearly hid the signal.
- [My Agents Were Calling Skills That Did Not Exist](agents-calling-skills-that-do-not-exist.md): the cleanup that turned a scattered pile of skills into something worth publishing.

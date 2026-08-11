---
skill: prep
type: on-demand
triggers: ["prep for", "before the meeting", "load context for", "meeting prep", "stakeholder prep", "run prep", "/prep"]
output: "Structured context brief for an upcoming conversation"
status: active
project: root
summary: "Load focused context before any stakeholder meeting. 2-minute pre-read ritual."
changelog:
  - 2026-05-20: Created. Vault-wide. Auto-routes by stakeholder, conflict-surfaces hypothesis vs decision gaps, structured post-meeting capture loop.
---

# Skill: Prep — Stakeholder Meeting Pre-Read

Load the 4 context layers that matter, surface any hypothesis-vs-decision conflicts, and produce a 2-minute brief before any stakeholder conversation.

---

## WHAT IT DOES

1. Routes to the correct project based on stakeholder name (no manual project lookup needed)
2. Loads 4 context layers: recent decisions · open hypotheses · open tasks · last session note
3. Surfaces conflicts where a live hypothesis contradicts a recent decision
4. Outputs a single structured brief — readable in under 2 minutes
5. Runs a post-meeting capture loop as a first-class ritual (not optional)

---

## HOW TO TRIGGER

**With a stakeholder name:**
- "prep for [name]"
- "meeting prep for [name]"
- "load context for the [meeting type] call"
- "/prep [name or meeting type]"

**With no arguments (dead-simple trigger):**
- "/prep" or "run prep" with nothing else

If triggered with no arguments, ask exactly one clarifying question before proceeding:
> "Who's the meeting with and what project?"

Wait for the answer, then run all steps.

---

## STAKEHOLDER ROUTING TABLE

The routing table is environment-specific and lives in `local-context.md`, alongside
this file. Read it and use it to auto-detect the project. Do not ask the user to specify
the project when the stakeholder maps cleanly there.

**If `local-context.md` is absent**, there is no routing table in this environment. Ask
the one clarifying question above and use the answer directly. Never guess a project
name, and never invent a stakeholder mapping.

**If a stakeholder maps to multiple projects** (rare), ask: "This meeting touches
[Project A] and [Project B], which is primary?"

---

## EXECUTION

### Step 1 — Confirm routing

Confirm the project from the routing table above. State it explicitly before loading anything:
> "Routing to: [Project], [Stakeholder role]"

---

### Step 2 — Load 4 context layers

Run these reads in parallel. All paths are relative to the project folder: `Projects/[Project]/`.

| Layer | File | What to extract |
|---|---|---|
| Decisions | `knowledge/decisions.md` | Last 3 durable decisions from the past 30 days. If none in 30 days, note "No recent decisions." Note provenance tag if present (`Verbal / 1:1 / async`). |
| Hypotheses | `knowledge/hypotheses.md` | All `status: proposed` and `status: candidate` entries. Extract: hypothesis text + confidence + next-test date. |
| Open tasks | Today's daily note (`01 Daily Notes/YYYY-MM-DD.md`) | Tasks tagged `#[project]` with `not done` status. Max 5. |
| Last session | `knowledge/log.md` | Last 3 lines (grep `^##`). Pull the most recent session date + one-line summary. |

If any file does not exist, skip that layer and note it as "Not found, context layer skipped."

---

### Step 3 — Conflict surface (core value step)

After loading decisions and hypotheses, run this check before writing the brief:

**For each `proposed` or `candidate` hypothesis:**
- Does it make a claim that a recent decision has already acted against?

**Flag pattern to look for:**
- Hypothesis claims X is the bottleneck: recent decision reduced investment in X
- Hypothesis proposes testing Y: decision has already locked against Y
- Hypothesis confidence is `high` but a decision was made without it being resolved

**If a conflict is found, output it as a CONFLICT block in the brief:**
```
⚡ CONFLICT: [H-XXX] says [claim]. But [date] decision [did the opposite / bypassed this]. 
Recommend: surface this in the meeting, either close the hypothesis or revisit the decision.
```

**If no conflicts found:** output "No hypothesis-decision conflicts detected."

---

### Step 4 — Output the brief

Output this exact structure. Keep it under 300 words total — the goal is a 2-minute read, not a summary document.

```
## Pre-read: [Stakeholder] / [Project], [YYYY-MM-DD]

**Meeting type:** [1:1 / review / planning / ad hoc]
**Time to read:** ~2 min

---

### Recent Decisions (last 3)
- [Date], [title]: [one sentence]. Provenance: [Verbal / 1:1 / doc if tagged].
- ...

### Open Hypotheses
- [H-XXX], [title]: [one-line claim]. Confidence: [low/medium/high]. Next test: [date or "overdue"].
- ...

### Open Tasks (tagged #[project])
- [ ] [task text]
- ...

### Last Session
[Date], [one-line summary from log.md]

---

### ⚡ Conflicts
[CONFLICT block(s) or "No conflicts detected."]

---

### Suggested focus for this meeting
[1-3 bullet points. Derive from the conflict(s), overdue hypothesis tests, or open decisions that need verbal confirmation. Never fabricate, only suggest what the loaded context supports. A hypothesis next-test is overdue if its next-test date is before today's date. Today's date = current date. Pull from context, not assumption.]
```

---

## POST-MEETING CAPTURE (required ritual, not optional)

After every meeting, run this capture loop. Do not skip. Do not make it optional. If the user says "done" or "meeting wrapped", prompt them immediately:

> "Quick capture, 3 questions, 30 seconds each:"
> 1. "Any verbal positions or commitments made? (e.g. 'the founder said the pricing page is a priority')"
> 2. "Any new evidence: data, user quotes, signals that affect an open hypothesis?"
> 3. "Any new open question or hypothesis that came up?"

Then route each answer to the correct file:

| Input type | File | Format |
|---|---|---|
| Verbal position / commitment | `knowledge/decisions.md` | Append under today's date. Provenance: `Verbal, [stakeholder], [date]`. |
| New evidence for a hypothesis | `knowledge/hypotheses.md` | Append to the matching hypothesis's `evidence:` list. Update the hypothesis's evidence count field in hypotheses.md directly. |
| New hypothesis | `knowledge/hypotheses.md` | Add under `## Active` with `status: candidate`, `confidence: low`, `evidence: []`. |
| Contradicts an existing hypothesis | `knowledge/hypotheses.md` | Add as counter-evidence to matching hypothesis. Consider status downgrade. |

After routing, output a 3-line confirmation:
```
Captured: [X] items routed
- decisions.md: [n] entries added
- hypotheses.md: [n] entries updated / added
```

---

## WORKED EXAMPLE, a hypothesis contradicted by a funding decision

This is the shape the conflict step is looking for. Names and numbers are illustrative.

**Trigger:** "prep for [the founder]"

**Step 1, routing:** the founder maps to [Product A] in the routing table.

**Step 2, load context:**
- Decision log: a decision dated 12 weeks ago committed spend to a standalone low-price
  product line, with no near-term investment in the pricing page.
- Hypothesis list: H-001, `status: proposed`, `confidence: medium`. Claim: "pricing page
  friction is the number one conversion bottleneck." Next test: funnel instrumentation,
  due 10 weeks ago.

**Step 3, conflict surface:** H-001 names the pricing page as the primary bottleneck.
The decision moved capital somewhere else entirely. The test that would settle it is
overdue and was never run, so confidence is still `medium` and the question is open. The
team is acting against a live hypothesis it never resolved.

```
CONFLICT: H-001 claims the pricing page is the number one bottleneck (medium confidence, unresolved).
A decision 12 weeks ago funded the standalone product line instead.
The deciding test (funnel instrumentation) is 10 weeks overdue and was never run.
Recommend: ask in this meeting whether pricing work is still on the roadmap or the new bet replaced it.
If the new bet replaced it: kill or defer H-001. If both are live: define which runs first.
```

**Step 4, brief output** follows the template above with the CONFLICT block surfaced first.

**Post-meeting capture:** if the founder says "focus on the new line for 60 days, pricing
is on hold", that verbal goes to the decision log with a verbal provenance tag, the
stakeholder, and the date. H-001 moves to `deferred`.

The generalisable point: a hypothesis nobody killed and nobody tested is the most
expensive object in a decision log, because the team keeps paying to route around it.

---

## NOTES

- This skill reads files — it never writes without the post-meeting capture prompt
- Conflict detection is pattern-matching on text, not semantic reasoning. Flag when uncertain and let the user confirm
- If hypotheses.md doesn't exist for a project, skip Step 3 and note "No hypotheses file found, conflict check skipped"
- If decisions.md is empty or only has old entries (>60 days), note "No recent decisions, consider logging from this meeting"

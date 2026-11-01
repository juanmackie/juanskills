---
name: on-track
description: "Check whether a project is still heading toward its owner's real outcome. Use when asked to run on-track, to check whether a project is on track or drifting, or to challenge its direction. Investigates evidence, grills the owner in rounds (Matt Pocock's grilling method) until the owner's decisions are settled, then drafts a 1-3 task recovery route."
version: 0.7.0
author: Juan Mackie
license: MIT
tags: [project-alignment, critical-review, grilling, wayfinding, outcome-validation]
metadata:
  hermes:
    tags: [project-alignment, grilling, decision-tree, outcome-validation]
    related_skills: [grilling, wayfinder, fable-loop]
---

# On Track

**Challenge the trajectory, not the task list.** The deliverable is a conversation that settles the owner's decisions, then a small, verifiable route. Never a status report with one question bolted on.

## Modes

- `run on-track on <project>` (default): investigate → grill in rounds across turns → route.
- `quick review`: findings and the recommended next move, no interview.
- "Give me the plan" (at invocation or any point mid-grill): stop asking, draft the route now (§5), and list every unsettled decision as a labelled assumption with your recommended default.

Read-only by default. Inspecting and drafting are fine. Editing code or project docs, tracker changes, deploys, side-effecting probes, spending, credential changes and dispatching implementation agents need the owner's explicit go-ahead.

## 1. Investigate — facts are your job

Read the minimum primary evidence to answer:

- **Destination:** who it is for, what job it does for them, what "working" means.
- **Reality:** what has actually been observed working end-to-end, versus claimed by code, tests, docs, release notes or agent reports.
- **Trajectory:** what current work is, and whether it moves the destination.
- **Risk:** the riskiest untested assumption.

Rules:

- Historical numbers and reports are leads, not measurements. Never present them as current.
- If a fact can be checked safely, check it; never ask the owner for something you can look up. While a lookup is still running, ask only the frontier questions that don't depend on it.
- Untested is not failed. No evidence that the owner used the product is not evidence that it failed for them.
- Don't invent targets (latency thresholds, accuracy bars) or call a proposed fix a root cause.
- Stop reading when more evidence would not change your questions.

## 2. Build the decision tree

- Map the owner-level decisions (goal, audience, priority, scope, order, acceptable risk, what "good enough" means) as a tree: each decision branches into the decisions that hang off it.
- The **frontier** is every open decision whose prerequisites are settled. Ask the whole frontier in one round. A question that depends on another still-open question waits for a later round.
- Delete any question where every plausible answer leads to the same action; recommend that action instead.
- Avoid false binaries. If two goals could be sequential (personal use, then wider release), ask about their order and the gate between them.
- If the frontier is empty after investigation, don't invent questions; go straight to §5.
- If the owner has asked for one question at a time, do that (the grilling method's supported opt-out).

## 3. Ask a round

The message contains only these parts:

1. **Settled line** (from round 2 on): one line, e.g. `Settled: Q1 → personal daily use first (you said).`
2. **One insight** (round 1, or when new evidence changes the picture): at most 3 sentences, saying how you know each claim (seen now / documented / you said / unverified). No verdict labels such as AT RISK, no audit.
3. **The frontier:**

```text
❓ **Q1** - **<title>**: <context; alternatives labelled A/B/C when there are several>. <closing proposal worded so "yes" = your recommendation, e.g. "Should the first milestone be A?">

➡️ <recommended answer, one-sentence reason, the trade-off>

---

❓ **Q2** - **<title>**: ...

➡️ ...
```

Rules:

- Number questions continuously across the session (if round 1 used Q1-Q2, round 2 starts at Q3). A number always means the same question.
- Before sending, check every ➡️ line: if agreeing with it would mean answering "no" to its question, reword the question.
- Then stop and wait. No next-round questions, keep/stop/change report, permission menu, or answers on the owner's behalf.
- Ask in plain text. Use a question tool (e.g. `ask_user_question`) only if the user or host requires one: one call, exact schema. If it fails validation, ask once in plain text; no retry variants. A tool title, option list, acknowledgement or empty result is not an answer. Don't re-ask in prose a question a working tool is already showing.

## 4. Advance on each reply

1. Map each answer to its Q number or unmistakable content. Settle only what the owner actually chose, keeping their caveats and new constraints. Your recommendation never becomes their decision by default.
2. A bare "yes" to a multi-question round accepts every recommendation in that round: record it as such in the Settled line (`Q1-Q2 → your recommendations (blanket yes)`) and continue. Ask one narrow check first only for a question where a wrong reading would be costly.
3. Unanswered questions stay open: carry them into the next round under the same number, shortened. Never re-ask a settled question.
4. Experience the owner reports ("I haven't tried it in the car") is a fact about them: record it as said and adjust the route. If they haven't used it, the first proof is a first real use, not a fix.
5. If the owner changes the destination, rebuild the tree from that branch and name the questions it drops.
6. Recompute the frontier and ask the next round. Repeat until the frontier is empty.

## 5. Land the route

When the frontier is empty, or the owner asks for the plan:

- **North Star:** the user-visible outcome and the success conditions the owner actually agreed.
- **Keep / stop / change:** the fewest corrections. Delete before adding.
- **First proof:** the cheapest real-world test of the riskiest assumption, with pass / fail / inconclusive. Mark any metric you proposed as proposed.
- **Next 1-3 tasks:** each with purpose, prerequisite, deliverable and an observable acceptance check. Separate research from agent-ready build work.
- **Assumptions:** every decision the owner did not settle, labelled.

Then ask once whether this captures what was agreed, before anything is enacted. Drafting needs no approval; acting does.

## Before sending any message

- Did I look up every fact I could, instead of asking?
- Is every question a real owner decision with settled prerequisites, and did I ask all of those?
- Does "yes" mean my recommendation for each?
- Is everything I call the owner's decision something they actually said?

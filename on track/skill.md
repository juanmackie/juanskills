---
name: on-track
description: "Review an existing project's direction against the owner's real goal. Investigate evidence, challenge unnecessary work, then use Matt Pocock-style grilling: a decision tree, rounds of independent questions, a recommendation for each, and dependent follow-up rounds until there is a shared, actionable recovery route."
version: 0.6.0
author: Juan Mackie
license: MIT
tags: [project-alignment, critical-review, grilling, wayfinding, outcome-validation]
metadata:
  hermes:
    tags: [project-alignment, grilling, decision-tree, outcome-validation]
    related_skills: [grilling, wayfinder, fable-method]
---

# On Track

**Challenge the trajectory, not the task list.** Determine whether work is taking the project toward the outcome its owner wants. Question requirements before optimizing them; identify what should be kept, deleted, simplified, proved, or changed.

**Default deliverable: a conversation that resolves decisions, followed by a short, testable recovery plan.** Not a status report with a token question attached.

## When invoked

- `run on-track on <project>`: investigate, state one provisional finding, then **start a real grilling session in rounds**. Continue across user turns; do not declare the exercise complete after round one.
- `quick review`: give the evidence-backed assessment and recommended next move without a forced interview.
- `make a plan`: draft a provisional route now, calling out unsettled decisions; use grilling if the owner wants to settle them.
- A review permits safe, read-only inspection, **not** coding, writes, tracker modifications, production probes with side effects, deployment, spending, or credential changes. Obtain permission for consequential actions before taking them.

## 1. Investigate facts before asking questions

Read only enough primary evidence to understand:

1. **Destination:** original motivation, intended users, actual job-to-be-done, agreed constraints, success condition.
2. **Reality:** what has been observed working end-to-end versus claimed by tests, docs, agents, or release notes.
3. **Trajectory:** current priorities, next milestones, and what each contributes to the destination.
4. **Risk:** the strongest untested assumption, avoidable work, or mismatch between effort and value.

Prefer the owner's statements and accepted decisions, current project docs, real observations, and relevant repo/product evidence. Historical reports are leads, not fresh measurements. When a fact can be checked safely in the environment, **check it instead of asking the owner**. Delegate targeted fact-finding to a subagent when one is available and it genuinely saves time; do not compulsively load adjacent skills or search every session.

Keep distinctions clear: `OBSERVED NOW`, `DOCUMENTED`, `USER-STATED`, `HYPOTHESIS`, `UNKNOWN`. An untested use case is a validation gap, not proof of failure. Don't invent numeric success criteria or mislabel a proposed fix as a proven root cause.

Stop investigating when more evidence would not change the questions or recommendation.

## 2. Build a decision tree, not a questionnaire

Use the **grilling** method from Matt Pocock's `grill-me` / `grilling` skills:

- Map the consequential unresolved choices as a **decision tree**. Every decision may create, remove, or alter decisions downstream.
- For each candidate question, identify its **prerequisites**, the recommendation you would make, and how plausible answers would change the route.
- The **frontier** is the set of all genuinely relevant decisions whose prerequisites have already been settled. Ask that frontier **together in one round**.
- Do **not** ask a downstream question while an upstream answer could invalidate it. Recalculate the frontier after every round; do not use a static checklist.
- Ask the owner to decide trade-offs, priorities, scope, value, and acceptable risks. Investigate technical facts yourself. Personal experience can be asked when it is not recorded or observable by the agent.
- Each question must be consequential: if every plausible answer leads to the same action, **delete the question** and recommend the action instead.

**Do not enforce one question per turn.** One question is fine if the frontier contains one; if there are three independent decisions, ask all three. Do not overwhelm the owner with irrelevant or premature questions merely to fill a batch.

## 3. Ask questions the way Grill Me does

Start a round with **one compact insight** from your investigation, ideally the discrepancy between engineering progress and actual outcome proof. Then ask the current frontier in this form:

```text
❓ **Q1 — <short title>**: <specific decision, context, and meaningful alternatives>

➡️ **My recommendation:** <clear proposed choice and why>. Would you go with that?

---

❓ **Q2 — <short title>**: <another independent decision>

➡️ **My recommendation:** <clear proposed choice and why>. Would you go with that?
```

- Number questions (`Q1`, `Q2`, ...) so the owner can answer `Q1 yes; Q2 option B` or simply explain in their own words. Use stable identifiers for the current round; don't recycle IDs for a different question in that round.
- Phrase each question so **"yes" accepts your recommended answer**. Offer a recommended option rather than presenting a blank-ended interrogation.
- Give the *reason* for the recommendation, including a material trade-off, in one or two sentences. Recommendations are **proposals**, not approved decisions.
- Separate questions with `---` when there is more than one.
- Put **only the current frontier** in the message; no speculative next-round questions, lengthy keep/stop/change report, or permission menu appended afterward.
- **Wait for the user's answers.** Never simulate the conversation, answer on the owner's behalf, or roll into the next round in the same reply.

### Question delivery and tool reliability

**Plain conversational text is the default**; Matt's questioning format does not require an `ask_user_question` tool. Do not call a question tool merely because one is available. If the user or environment explicitly requires one, follow its exact schema and use only one presentation channel.

If a question tool fails validation, fall back to plain text once; do not try multiple malformed variants. If it returns an **actual user answer**, consume it and advance. A tool label, displayed options, acknowledgement, empty result, or status is **not** a user answer. Never manufacture `USER-STATED` evidence from a question title. Do not ask the same question in prose if a functioning tool already presented it and awaits an answer.

## 4. Each answer advances the tree

On the next user turn:

1. **Parse only the answers actually supplied.** Match them to the current `Q` identifiers or unmistakable question content. `Yes` to a single question accepts its recommendation. For a multi-question round, an unqualified `yes` is ambiguous unless the user clearly means **all** recommendations; clarify only the affected interpretation.
2. **Mark a decision settled** only when the user actually chose or accepted it. Preserve corrections, caveats, and newly stated constraints. Do not transform the agent's earlier recommendation into a user decision.
3. **Update the route and dependencies.** A settled answer may eliminate entire branches or unlock new questions. Explicitly drop invalidated branches.
4. **Recompute the frontier** and ask the next round of independent, ready decisions, with a fresh recommendation for each. Refer briefly to what changed; don't reprint the original audit.
5. **Never repeat an answered question.** If the user skipped Q2, keep Q2 open rather than asking Q1 again; clarify an ambiguity with a narrower question only when it matters.
6. **Stop grilling when the frontier is empty**: all consequential owner decisions have been explored, or remaining gaps are researchable facts to investigate. Do not keep questioning for ceremony.

If the user asks to stop grilling or just give the plan, comply and mark the unsettled decisions as assumptions. If the user changes the destination, revise the tree instead of insisting on an older documented North Star.

## 5. Land on an actionable, minimal route

Once the decision frontier is empty, summarize the **shared understanding** and propose:

- **North Star:** user-visible result and genuinely agreed success conditions.
- **Keep / stop / change:** the fewest meaningful corrections. Delete needless work before adding new work.
- **First proof:** the cheapest real-world test of the riskiest assumption; what would constitute a pass, failure, or inconclusive result, with proposed metrics labelled as proposed.
- **Next 1–3 tasks:** each with purpose, prerequisites, concrete deliverable, and observable acceptance evidence. Separate unresolved research from agent-ready implementation.

Ask the owner to confirm that this accurately captures the decisions **before enacting** it. Drafting is allowed; changing canonical docs, dispatching agents, creating tickets, editing code, deploying, or otherwise modifying project/external state needs the applicable authorization. Don't seek a ritual confirmation just to *reason* or *draft*.

## Voice-MVP regression scenarios

**R1 — Avoid a status-report dead end.** After reading a voice project, do not finish with "AT RISK. Is the North Star still right?" Identify the real user-experience evidence gap, then open the current decision frontier.

**R2 — Two independent choices.** If target user and priority use environment are both open and neither depends on the other, ask **Q1 and Q2 in the same round**, with a recommended answer for each. Don't ask one and terminate, or ask about latency targets before knowing which experience matters.

**R3 — Dependent next round.** If Q1 decides that the first milestone is personal daily use, only *after* Q1 is settled ask about that milestone's acceptance bar. Do not preselect `p50 < 3s` as if the owner agreed.

**R4 — No invented lived experience.** A repo lacking a phone pilot does not mean the owner tried it and stopped. Ask what they've actually experienced if it affects the route; accept "haven't tried it" as a valid answer.

**R5 — No duplicate answer handling.** A question-tool validation error does not equal a user response. A valid tool selection should advance the tree, not cause the same question to be printed again.

**R6 — No forced choice between sequential objectives.** "Personal use or wider release?" may be a false binary. Ask about order and release gate instead, with a recommendation.

**R7 — No endless interrogation.** If all remaining unknowns can be established by safe checks, investigate them and draft the route. When the frontier is empty, summarize and seek confirmation before implementation, not another arbitrary question.

**Final test:** Did I identify the real destination, separate facts from decisions, ask every *ready* consequential question with a recommended answer, advance from the user's actual answers, and arrive at the smallest verifiable route without silently acting on it?

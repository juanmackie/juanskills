---
name: on-track
description: "Challenge whether an existing project is still solving the right problem, find drift, and drive a focused conversation toward a simpler, testable next move. Use for 'are we on track?', stalled/overbuilt projects, milestone reviews, and reorientation. Investigate evidence, give a pointed recommendation immediately, then grill consequential decisions one question at a time."
version: 0.3.0
author: Juan Mackie
license: MIT
tags: [project-alignment, drift-detection, decision-grilling, outcome-validation, recovery-planning]
metadata:
  hermes:
    tags: [project-alignment, drift-detection, decision-grilling, outcome-validation, recovery-planning]
    related_skills: [grilling, domain-modeling, wayfinder, fable-method]
---

# On Track

**Do not give me a status report when I need a course correction.**

Your job is to decide whether the project's *present trajectory* can deliver the outcome the user wants, find the highest-leverage correction, and have a sharp conversation that turns that correction into an executable next move. Be a skeptical, constructive project partner: investigate first, take a position, challenge its assumptions, and ask the human only what the evidence cannot decide.

A successful invocation is not a polished audit. It is a **better decision, a smaller route, and a clear next proof of value**.

## Trigger and modes

Use for an *existing* project when the user asks to check alignment, momentum, direction, scope, priorities, or what to do next. Not for ordinary coding or a new idea with no history.

- **Default: ACTIVE REVIEW.** Inspect, offer a substantive diagnosis and suggested action, then start a one-question-at-a-time grill. Do not stop just because the original North Star is technically unconfirmed.
- **QUICK AUDIT:** If the user explicitly asks for a brief verdict or no interview, provide a compact assessment and one recommended next action. Do not force questions.
- **RECOVERY PLAN:** If the user explicitly asks for a plan, give the best evidence-backed *draft plan now*, with assumptions marked. Ask only questions that would materially alter it. Approval is required for actually changing artifacts, not for proposing a plan.

## Operating principles

1. **Outcome over activity.** Tests, releases, tasks and agent reports are evidence about delivery, not proof of user value. Assess both engineering maturity and outcome validation.
2. **A recommendation is not a commitment.** You may infer a provisional destination, challenge it, and recommend a better route without permission. The user alone accepts changes to goals, scope, priorities, or canonical plans.
3. **Facts are investigated; choices are grilled.** Look in the workspace before asking factual questions. Ask the user about purpose, trade-offs and preferences, not something a safe test could answer.
4. **Subtract first.** Challenge the requirement; delete nonessential work; simplify what remains; only then optimize or automate. Consider the opportunity cost of continuing.
5. **No artificial gates.** Do **not** ask “Is this still the North Star?” merely because the file says the goal is provisional. Ask for clarification only if competing interpretations would change the next move. Never treat a proposal as an approved decision.
6. **Read-only diagnosis, explicit write approval.** Read files, inspect git and run safe, nonmutating checks. Do not edit project files, create tickets, change branches, commit, deploy, send messages, spend money or run destructive/prod-impacting probes without explicit permission. Drafts in chat are fine.
7. **Challenge, don't rubber-stamp.** State what you think should be stopped, simplified, tested, or reconsidered. Show disconfirming evidence. Don't manufacture disagreement or invent certainty.
8. **Tools earn their keep.** Load only the skills and evidence necessary for the decision; do not ritualistically load adjacent skills, search every transcript, or repeat facts already verified. Do not pretend to have invoked a missing skill.

## Phase 1: Find the project and the real bottleneck

Inspect just enough primary evidence to answer:

- **WHY:** What user pain, actual user, intended experience, and desired result justify this project? Cite the source.
- **NOW:** What demonstrably works today? What has only been claimed, simulated, or tested by proxy?
- **NEXT:** What work is currently underway, and which remaining milestone allegedly unlocks value?
- **GAP:** Which missing capability, assumption, or real-world proof most threatens the outcome?

Practical investigation order:

1. Current user instruction and the most relevant accepted goal/decision document.
2. Project README/plan and *one* recent status or release artifact.
3. A focused, safe reality check (e.g., a relevant test, local state, known user observation) **only when it changes the decision**.
4. Expand only to resolve contradictions or a decisive unknown. Prefer primary artifacts over memory and agent reports.

Use compact evidence labels only for load-bearing points: `OBSERVED`, `DOCUMENTED`, `REPORTED`, `INFERRED`, `UNKNOWN`. A release can be `OBSERVED` and its real-world usability can remain `UNKNOWN` at the same time. Record file paths, commands, commits or links as references. Do not turn the response into a forensic inventory.

**Stop searching when additional artifacts are unlikely to change the recommendation.** Do not invoke sibling skills just because they are listed in frontmatter. Consult `grilling` or `domain-modeling` only when needed and available; this skill is self-contained.

## Phase 2: Attack the project's central assumption

Trace the shortest causal chain:

`User problem -> intended behaviour -> observed proof -> remaining gap -> next discriminating step`

Interrogate it internally before talking to the user:

- If the present plan succeeds exactly as written, does the user actually get the desired outcome?
- Which part is genuinely critical versus elaborate work undertaken because it is interesting or already started?
- What has been demonstrated **in the real use case**, not just in tests, docs, or a release log?
- What must be true for the current approach to beat a simpler alternative, a manual process, or doing nothing?
- What evidence would change your recommendation? What cheap experiment would provide it?
- What should be **stopped or deferred now** to accelerate useful learning?

When a fork is observable (reliability, behaviour, latency, usability, performance), recommend a test/prototype rather than asking the user to guess. When it is a value choice (who the product is for, privacy trade-off, acceptable experience), grill it.

**Do not equate “not validated yet” with “we know nothing.”** You may have enough evidence to say the technical trajectory looks healthy *but product validation is the critical risk*.

## Phase 3: Give a useful answer BEFORE the interview

The first response in an ACTIVE REVIEW must include:

1. **My read of the goal:** one sentence; mark `provisional` only when necessary. Do not demand confirmation as a ritual.
2. **My verdict:** a plain-language conclusion about direction, with the most important uncertainty. You may distinguish `BUILD PROGRESS` from `USER-OUTCOME PROOF`.
3. **The uncomfortable finding:** one real mismatch, false proxy, scope trap, or assumption, supported by evidence.
4. **My proposed correction:** an actionable, *read-only* draft of what to keep, stop, change and verify. Give a **specific next experiment or deliverable**, success/failure signals, and how the results change the route. Do not wait for permission to think.
5. **One consequential question:** the best decision fork the evidence cannot resolve, with your recommended answer and why. Then wait for the reply.

Prefer roughly 200–400 words for this first response. Avoid a giant checklist, a rigid machine-looking status dump, or an overwhelming backlog. A user should be able to act on the recommendation even without answering.

**Bad:** “INSUFFICIENT EVIDENCE. Is this still your North Star, and should we test it?”

**Better:** “The backend appears to work, but we're using release verification as a proxy for the hands-free experience. I would pause new features and run a short real-phone pilot that tests pause, interruption, Stop and background results. If it fails, fix interaction reliability before expanding scope; if it passes, start daily-use trials. My question: is the next target *your own dependable daily use* or *a product ready for other users*? I'd optimise for your daily use first because it gives faster, clearer feedback.”

Do not claim a pilot passed, an acceptance bar was approved, or a root cause was proven unless evidence supports it.

## Phase 4: Grill the real decisions, one at a time

After each user answer:

1. Restate the **decision actually settled** in one sentence; don't repeat the entire audit.
2. Update the working goal, constraint, or decision map. Distinguish user decision from agent recommendation.
3. Ask the **next highest-impact unresolved question** with an opinionated default, *only if its answer changes the plan*.
4. If an answer reveals a testable factual uncertainty, investigate/propose a probe rather than making the user speculate.
5. When no material forks remain, **stop interviewing** and present the revised plan. Do not create a never-ending grill.

Useful question pattern:

> “We can prioritise **A** or **B**. I recommend **A**, because [consequence]. The trade-off is [cost]. Which should govern the next milestone?”

Order questions by dependency (user and desired outcome before product scope; scope before architecture; architecture before implementation). Challenge answers respectfully if they conflict with evidence or the user's stated outcome. Do not ask for mere affirmation of an already strong working assumption.

Only record durable jargon and consequential hard-to-reverse decisions. Suggest a glossary or ADR when genuinely useful; do not scaffold documentation by default, and do not write it without approval.

## Phase 5: Produce a minimal route, not another backlog

As soon as enough decisions are settled, produce a **proposed** recovery route:

- **Destination:** agreed user-visible result and limits; clearly mark any remaining assumptions.
- **Keep / stop / change:** what survives, what is cut or paused, and why.
- **Next proof:** the smallest real-world experiment or vertical slice, exact observation and a meaningful pass/fail criterion. Where metrics are not established, propose criteria; don't pass them off as approved.
- **Branching outcome:** what to do if the proof passes, fails, or remains inconclusive.
- **Small agent queue:** ideally 1–3 bounded tasks with goal link, inputs, exclusions, deliverable and independently checkable acceptance condition.
- **Human decisions:** only the remaining choices requiring human judgment.

When the route spans multiple uncertain sessions, use **Wayfinder ideas**: name the destination, distinguish decisions already made from unresolved fog, identify prerequisites and the next takeable decision frontier. Do not generate tickets for fog that cannot yet be specified. Decision tickets are not implementation tasks. Do not start a full issue-tracker ceremony when a short plan is sufficient.

**For approved implementation**, hand off to the project's existing execution flow. For unapproved plans, give a copyable proposed handoff without modifying state. Request specific approval once, at the point of a requested write or execution action; do not require two generic approval rituals.

## Verdict guide (not a forced template)

- **ON TRACK:** The goal is coherent, outcome evidence is meaningful, and the remaining route is credible.
- **AT RISK:** There is meaningful progress, but a decisive assumption or outcome proof is unresolved. Name the next discriminating test.
- **OFF TRACK:** Work is diverging from the intended result, or the route is contradicted by evidence. Recommend what to stop.
- **INSUFFICIENT EVIDENCE:** You cannot responsibly assess *the trajectory* from available evidence. State the minimum missing observation and still propose how to obtain it. Do not use this label merely because no user pilot exists when that absence itself establishes the key risk.

If the destination is ambiguous, label the assessment `PROVISIONAL` and ask the *distinguishing* question, not a generic confirmation question. A provisional goal does not prevent a provisional read-only recommendation.

## Worked calibration: voice-interface project

If the repository supports the thesis “voice for hands/eyes-busy use,” and an API/release has been verified but no real-device user pilot is evidenced:

- Acknowledge that shipping and background-result delivery are engineering progress.
- Identify the untested **actual experience** as the key risk; do not dismiss existing work.
- Suggest a small real-phone study with representative interruptions, silences, Stop and asynchronous results; keep stated privacy constraints intact.
- Propose success observations and different next actions for pass/fail/inconclusive outcomes.
- Ask a product-purpose fork, such as **personal daily-use tool vs wider-user product**, if that choice isn't settled. This is more useful than merely asking whether the original North Star still applies.

## Quality check before every turn

- Did I make a **judgment** and propose a concrete move, rather than just summarise evidence?
- Am I confusing passing tests with solving the user's problem?
- Did I cut or challenge at least one assumption or unnecessary step where justified?
- Have I separated measured facts, documented claims, and my interpretations?
- If asking a question, is it a **human decision** whose answer changes the next step? Is my recommendation attached?
- Did I avoid redundant skill loads, unnecessary searches and ceremonial approvals?
- Did I preserve read-only diagnosis, and avoid modifying project state without explicit authorisation?

**Done means clarity and leverage, not merely the existence of an audit.**

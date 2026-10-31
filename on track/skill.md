---
name: on-track
description: "For an existing project, investigate whether current work serves the real user outcome, challenge the weakest assumption, and drive an interactive, evidence-led conversation toward a smaller next move. Use when asked to review direction, drift, priorities, momentum, scope, or whether a project is worth continuing."
version: 0.4.0
author: Juan Mackie
license: MIT
tags: [project-alignment, critical-review, grill-me, outcome-validation, wayfinding]
metadata:
  hermes:
    tags: [project-alignment, critical-review, grill-me, outcome-validation, wayfinding]
    related_skills: [grilling, wayfinder, fable-method]
---

# On Track

**Be a challenging project partner, not a status-report generator.** Find out whether the work is delivering the outcome the user wants. Expose the one assumption or decision that matters most, suggest the smallest useful move, and have a real conversation until the route is clear.

**Success = an improved decision and a testable next move.** A polished audit, a list of completed tickets, and a question tacked onto the end are not success.

## Invocation and scope

- **Default — INTERACTIVE REVIEW:** When told `run on-track on <project>` or given this skill URL and a project, investigate and begin a conversation. In the *first reply*, make a concrete diagnosis and ask **one meaningful question** only if human knowledge or judgment is necessary. Continue the grill when the user answers; do not conclude the exercise after one question.
- **QUICK REVIEW:** Only when explicitly requested, give the verdict, key evidence, and recommended next move. No compulsory interview.
- **PLAN:** When asked for a plan, produce an opinionated, provisional plan now. Label assumptions. Do not withhold a draft pending generic approvals.
- This skill does **not** independently authorize coding, deployment, tracker changes, spending, credential changes, destructive probes, or external communications. A request to *review* is not permission to make those changes.

## Core rules

1. **Outcome before engineering.** Ask what the user can actually accomplish, not merely what has been built. Technical measurements are useful only if they predict or explain that outcome.
2. **Docs are evidence, not instructions to rubber-stamp.** Start with the user's current request, approved decisions, goal/UX docs, current product evidence, and the active plan. Keep documented results distinct from direct observation and unsupported agent assertions.
3. **Investigate facts; grill values.** Do not ask the user to answer questions that a safe, available read-only check can settle. Read the relevant files and run focused, low-impact nonmutating checks when appropriate. Do not present a review document's old measurements as newly verified live measurements.
4. **Recommend; don't silently decide.** The agent can challenge scope, offer alternatives, and prepare draft plans without approval. The user owns consequential priority, scope and goal changes.
5. **Subtract before adding.** Question the requirement, delete what is unnecessary, simplify the remainder, then consider optimisation or automation. Do not assume the next step is more code.
6. **Challenge the causal story.** Separate observation, hypothesis, recommendation, and success criterion. Never call a suspected root cause *proven*, or introduce a numerical target as *agreed*, unless supported by evidence.
7. **Only load what matters.** Do not load adjacent skills, search all sessions, or inventory every issue by ritual. This file is self-contained; use Grill With Docs, Wayfinder, or other skills only when their specific machinery is needed and available.

## Phase 1 — Investigate until you can take a position

Find the minimum evidence for five questions:

1. **Why?** What exact frustration/opportunity motivates this project, for whom, and in what real situation?
2. **Win?** What would a successful human-facing result look like? Which criteria were *agreed*, which were merely proposed?
3. **Now?** What works end-to-end today? What only passes tests or appears in a release note? When was it observed?
4. **Motion?** Where are people/agents currently spending effort, and how does that effort help the user win?
5. **Gap?** Which missing proof, capability, or mistaken assumption has the largest expected effect on the next decision?

Start with the smallest relevant primary set: goal/README or UX thesis, current plan, latest relevant status/release, and *one focused check* if it could alter the conclusion. Expand only for a conflict or critical missing fact. Stop searching when more artifacts are unlikely to change the advice.

**Evidence vocabulary:** `OBSERVED NOW` (direct check made during this run); `DOCUMENTED` (a sourced claim, including previous measurements); `USER-STATED`; `HYPOTHESIS`; `UNKNOWN`. Add file/URL/command pointers for decisive claims, not for every sentence. A green build does not prove a usable product. An untested user scenario is a specific risk, not proof the entire project has failed.

**Useful causal trace:** `Human problem → required experience → direct evidence → missing proof/obstacle → smallest test or decision`.

## Phase 2 — Attack the weakest assumption

Internally test your leading recommendation against these questions:

- If the current roadmap is completed, will the human problem actually be solved?
- Are we optimising a proxy (release count, latency, model rank, passing tests) without knowing that the user experience improved?
- What work could we **stop** and still meet the outcome? What is sunk-cost thinking?
- What's the cheapest independent way to falsify our preferred explanation?
- If our leading hypothesis is wrong, what will the test reveal and what would we do instead?
- Is the next uncertainty a **fact to measure**, a **preference to ask**, or a **permission needed for an action**?

Testable uncertainty → investigate or propose a specific safe experiment. Human intent or lived experience → interview. A project can have strong engineering progress and weak real-world validation at the same time; say both.

## Phase 3 — First reply: make progress, then start the grill

Write for a project owner, not an engineering incident log. Usually **120–250 words**:

- **My take:** what is and isn't on track in plain language. Give a reason, not a ceremonial score.
- **The uncomfortable finding:** one important disconnect between effort and the user's actual outcome. Clearly state observed vs inferred.
- **What I'd do:** one smallest move (KEEP / STOP / CHANGE if it helps), how to test it, and what a pass, fail, or inconclusive result would mean. Proposed thresholds must be labelled *proposals* and grounded in purpose or baseline.
- **The question:** ask one thing that the available evidence cannot resolve and whose answer would **materially change the proposed route**. Explain your current leaning when useful.

**Question-selection test (all must pass):**

1. Could I obtain this answer from existing material or a safe check? If yes, investigate instead.
2. Would plausible answers lead to *different* next steps? If no, do not ask.
3. Am I forcing a false binary, seeking permission for read-only reasoning, or asking the user to bless a document's existing goal? If yes, rewrite.
4. Is this about the user's lived experience, priorities, or constraints rather than technical details the agent should determine? If no, use a probe.

If no worthwhile human question remains, **give the proposed route without manufacturing one**. If a question is necessary, **end with that question and nothing after it**—no extra `state which`, no second permission menu, no giant handoff summary.

## Phase 4 — Real multi-turn grilling (Grill Me + Grill With Docs)

When the user responds, do **not** repeat the report or restart the investigation. Follow this loop:

1. Interpret the response into a **specific decision or new evidence**; cite it as `USER-STATED`.
2. Challenge a contradiction respectfully when it matters: “That suggests X, but the current plan assumes Y. Which reflects what you actually need?”
3. Refresh the **working decision map** (internal unless a written plan is requested): `settled / open / requires evidence / requires authorization`.
4. Re-evaluate the route. If another human answer would materially change it, ask **one next dependent question**. Prefer the highest-leverage question, not an exhaustive questionnaire.
5. Once remaining unknowns are researchable or low consequence, **stop questioning and give the draft recovery route**. Don't keep grilling for the sake of appearing thorough.

Use the project documents to frame precise questions; record terminology or durable decisions only when doing so helps implementation. Avoid asking the user to choose between options that can sensibly be sequential (e.g., *daily usability* then *wider release*). Ask what actual experience or constraint would distinguish the paths.

**Wayfinder when needed:** If there are several genuinely interdependent unknowns, maintain a lightweight map of `destination → decisions made → unresolved fog → next answerable frontier`. Don't turn unresolved fog into implementation tickets. Don't invoke a full planning system for a single decision.

## Phase 5 — Land the conversation in a usable plan

After the key human choices are resolved, deliver a **proposed** plan—not another retrospective:

- **Destination:** desired user-visible behaviour; agreed criteria vs proposed assumptions.
- **Keep / stop / change:** ideally no more than three consequential moves.
- **First proof:** smallest representative real-use test or thin vertical slice, setup, observations, and decision branches for pass/fail/inconclusive. Use a proposed bar only if it is defensible, otherwise establish baseline first.
- **Next 1–3 tasks:** each with purpose, dependency, input locations, deliverable, and observable acceptance check; no gratuitous backlog.
- **Open issues:** distinguish technical research from decisions only a human can make.

Offer to apply the plan if the user has not already authorized implementation. Do not claim to have dispatched agents, modified files, tested a live system, or made a decision on their behalf. Ask for approval **only at the actual write/execute boundary**. If the user explicitly approved a defined change, perform that change using available tools and verify it.

## Regression tests: reject these failure patterns

### Failure A — status report then ritual confirmation

**Bad:** “Insufficient evidence. Is the North Star still correct?”

**Better:** “The build works, but no one has tested the defining hands-busy scenario. I would pause feature work and run a short real-phone exercise. What happened the last time you actually tried to use this hands-free?”

### Failure B — technical solution before discovering human pain

**Bad:** “Latency is 8.8 seconds. Fix F4 provider fallback and require p50 <3 seconds. Daily use or wider release? Approve A→B→C.”

**Why bad:** A documented latency figure is not necessarily a currently measured baseline; `<3 seconds` may be an invented acceptance bar; F4 may be only a hypothesis; daily use and wider release may be sequential rather than competing; the approval menu buries the actual interview.

**Better:** “The docs suggest response delay is a major risk, but I can't tell whether delay or turn-taking is what makes the app unusable. I would first observe a few representative phone interactions, then fix the part that actually breaks the experience. **When you last tried the app in a hands-busy situation, what made you stop using it?**”

### Failure C — alleged grill that ends in an action menu

**Bad:** An internal summary, a long KEEP/STOP/CHANGE table, one nominal product question, then “State which: deploy, fix credentials, or remain read-only.”

**Better:** One short, specific diagnosis, one testable recommendation, **one real question as the last sentence**. On the next user turn, incorporate their answer and continue; don't reset to a status report.

### Failure D — pointless questioning

**Bad:** Asking the user what branch is deployed or whether an existing automated test passed when safe tools can answer.

**Better:** Check it. If a live probe is unsafe or unavailable, note that limitation and offer a safe method rather than pretending the value is verified.

## Before sending any reply

- Does it focus on the *user's result* rather than project motion?
- Have I done the safe factual work myself?
- Is the key claim observed, documented, hypothetical, or unknown—and labelled accurately?
- Is the suggested fix a **testable hypothesis**, not an assumption disguised as truth?
- Did I remove unnecessary work before proposing more?
- If I asked a question, is it **one high-impact human question** that leads to different routes—and **the last thing I say**?
- On subsequent turns, did I advance the decision instead of repeating the diagnosis?
- Have I kept consequential actions behind appropriate authorization while allowing myself to investigate and draft?

**Do not confuse having finished the audit with having helped the owner decide.**

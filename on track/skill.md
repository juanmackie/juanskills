---
name: on-track
description: "Check whether a project is still heading toward its owner's real outcome. Use when asked to run on-track, to check whether a project is on track or drifting, or to challenge its direction. Investigates evidence, grills the owner in rounds of numbered questions until the owner's decisions are settled, then drafts a 1-3 task recovery route."
version: 0.8.3
author: Juan Mackie
license: MIT
tags: [project-alignment, critical-review, grilling, wayfinding, outcome-validation]
metadata:
  hermes:
    tags: [project-alignment, grilling, decision-tree, outcome-validation]
---

# On-Track

You are running **on-track**: a short, calibrated interview that checks whether a
project is heading where its owner thinks it is, then turns the answer into a
minimal route of 1–3 agent-ready tasks.

The deliverable is **shared understanding first, route second**. Do not act on
the project until the owner confirms the route.

## When to use
- "Run on-track on this repo."
- "Is this project on track?"
- "Review the direction and tell me what to do next."
- A project has activity (commits, plans, modes) but no proof it is working.

## When not to use
- The owner wants a code review → use a review skill.
- The owner wants a spec → use to-spec.
- The owner wants a bug fixed → diagnose first.

## Hard rules (these prevent the known failure modes)

1. **Questions are your reply.** Write them in chat. Never send them through a
   tool, `echo`, or a script.
2. **One question at a time.** Ask, then stop. Wait for the answer before
   computing the next question. If the owner explicitly asks for a batch, you may
   send up to 3 *independent* questions in one round, each numbered with a
   recommendation — never more.
3. **Wait line only.** After a question, end the turn with a single short line
   (e.g. "Reply with a letter, or tell me in your own words."). Do not keep
   working, do not narrate compliance, do not pre-compute the route.
4. **Facts are your job, not the owner's.** Look up anything you can look up
   yourself. If a fact needs an active probe, dispatch a sub-agent with a fresh
   context so the round is not blocked — only the downstream question waits.
5. **Read to investigate; do not run the build.** Read code, docs, state, logs,
   and git history. Do not run test suites or builds to investigate. Running the
   evaluator is a *task in the route*, not investigation.
6. **Teach the frame when needed.** If the owner is learning the domain, teach
   the missing frame in 1–2 lines, then ask the next useful question. Do not
   over-grill basics.
7. **Stop early when clear.** If the plan is clear enough to draft a route, stop
   asking. The interview is a means, not a procedure.
8. **No acting before the gate.** No code edits, no long commands, no doc changes
   until the owner accepts the route and approves docs.

## The loop

### Phase 0 — Calibrate (one question)

Get the owner's calibration before grilling. You may infer from the repo and
*state your assumption* in one line, but give an easy override:

> Before I grill you: how well do you know this project — **New / Working /
> Expert**? And how hard should I push — **Light / Standard / Hard**?
> (I'll assume Working/Standard unless you say otherwise.)

Adapt:
- **New** → teach frames, gentler, explain why each question matters.
- **Working** → default pace.
- **Expert** → skip basics, sharper, push on evidence and second-order effects.
- Owner can say `softer`, `harder`, `teach more`, `skip basics` at any time;
  obey immediately.

### Phase 1 — Investigate (minimum primary evidence)

Read only what you need to build the decision tree:
- README / top-level docs.
- Recent git history (last ~5–10 commits).
- Project state dirs (e.g. `.dream-rsi/`, `state/`, `evaluators/`, `verification/`).
- The skill's own claims vs. what the repo actually contains.

Produce one short **Settled / Where we are / Gap** block. Do not dump the
investigation. Do not ask the owner for facts you could read.

### Phase 2 — Build the frontier

From the evidence, list the **open decisions** that actually block a route. Keep
only questions whose answer changes the route. For each:
- the decision,
- 2–3 concrete options,
- your recommendation + one-line reason.

Discard questions that are (a) answerable from evidence, (b) nice-to-have, or
(c) already settled.

### Phase 3 — Grill (one question at a time)

Ask the highest-value frontier question:

> ❓ **Q** — *[short label]*
> [question]
> A) …  B) …  C) …
> ➡️ Recommend **X** because [one line].

Then stop with a one-line wait line. When the answer arrives:
- Record it in a running **Settled** line.
- Recompute the frontier: some questions die, new ones may appear.
- Ask the next single question.

Repeat until the frontier is empty or the route is clear enough to draft. If an
answer opens a new branch, follow it — that is the point of one-at-a-time.

### Phase 4 — Draft the route

When the frontier is empty or clear, draft the route and ask:

**"Does this capture what we agreed?"**

Each task must be executable by an agent without further questions: a concrete
deliverable, a prerequisite, and a pass/fail check.

### Phase 5 — Accept, then docs

- Get an explicit **yes** on the route. If the owner says no, return to Phase 3
  with the objection as the new frontier.
- Then ask the docs question: **"Should I update project docs? Name files and
  changes."** List exact files and edits. Only edit what the owner approves. Do
  not edit state/advisory files unless asked.

## Output contract

- **Settled line** after each answer: `Settled: Qn → [answer].`
- **No narration.** Do not say "I will now…" or "As instructed…". Just ask.
- **Recommendation on every question.** Never ask a bare question.
- **Assumptions labeled** `proposed`; never presented as decisions.
- **Confidence + unverified** at the end: state what is proven and what is still
  unrun (e.g. "evaluator not yet run — that is task 1").
- **Turn ends on a question or the final report.** No work between asking and the
  answer.

## Question ladders (inline reference)

Use these angles to generate frontier questions. Pick only those that change the
route.

**Goal fit**
- What result must this project deliver — and in what order?
- Is the current work serving that result, or a proxy for it?
- What would make you say "this was the right thing to build"?

**Evidence & proof**
- What is *proven* working vs. only *claimed* working?
- What is the smallest verifiable result that would count as progress?
- What is still unrun, and what would running it cost?

**Constraints & reality**
- What is the real budget (time, spend, attention) for the next step?
- What must not break while we do this?
- What is reversible vs. one-way-door?

**Trajectory**
- What should we **stop** doing?
- What is the one behavior change that unblocks the north star?
- If this keeps going as-is for a month, what does it look like?

**Second-order effects**
- If this succeeds, what new problem does it create?
- Who or what depends on the current behavior?
- What are we optimizing that we should not be?

## Failure modes to avoid (observed)

- Asking questions through a tool → they vanish from the owner's view.
  **Write them in chat.**
- Continuing to investigate or draft after asking → the answer arrives too late.
  **Stop.**
- Asking the owner for facts you could look up → wasted round. **Look it up.**
- Sending 4+ questions at once → Q1's answer can't shape Q2. **One at a time.**
- Skipping calibration → wrong pressure for the owner. **Calibrate first.**
- Treating the route as the deliverable → the owner never confirms shared
  understanding. **Gate on yes.**

## Done criteria

The run is done when:
1. The frontier is empty (or the owner stopped early),
2. The route is accepted with an explicit yes,
3. The docs question is answered and approved edits are made (or explicitly
   declined),
4. The final report states: decisions settled, route, assumptions, and the first
   unrun proof.

Waiting line not needed once the owner has answered — end the turn.

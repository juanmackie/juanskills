---
name: on-track
description: "Audit whether a project is still advancing its intended outcome. Use for 'are we on track?', project drift, stalled projects, milestone reviews, agent work alignment, or replanning an existing project. Reconstruct the North Star from primary evidence, confirm it with the user, challenge the current route, and propose a minimal evidence-backed recovery plan."
version: 0.2.0
author: Juan Mackie
license: MIT
tags: [project-alignment, project-audit, drift-detection, replanning, agent-handoff]
metadata:
  hermes:
    tags: [project-alignment, project-audit, drift-detection, replanning, agent-handoff]
    related_skills: [fable-method, lifeos-loop, domain-modeling, grilling]
---

# On Track

**Audit the destination, not the activity.** Determine whether verified work is moving an existing project toward the outcome its owner actually wants. When it is not, expose the smallest consequential decisions and propose the shortest credible recovery route.

A green test suite, busy agents, closed tickets and completed milestones can all coexist with a failed project. The measure is **demonstrated progress toward the user-confirmed outcome**.

## Use this when

- The user asks whether a project is on track, drifting, stalled, overbuilt or worth continuing.
- A project has accumulated plans, pivots, code, handoffs or agent-generated tasks and needs an alignment review.
- The user wants a periodic review that converts a strategic objective into a few useful agent tasks.

For a single well-scoped implementation task, use the normal execution workflow. For an initial idea with no project history, use a planning/interview workflow. This skill **reviews an existing trajectory**; it is not a new project generator, coding skill or generic status report.

## Operating contract

1. **The user owns the North Star.** Reconstruct a *candidate* from evidence, then have the user confirm or correct the destination and material success criteria. Do not silently replace the objective with what the agent thinks is better.
2. **Facts are investigated; decisions are asked.** Search accessible primary sources before questioning the user. Keep observed facts, user commitments, agent claims and inferences distinct. Do not let an agent-generated plan authenticate itself.
3. **Diagnosis is read-only.** Inspect artifacts, issue trackers, code, history and safe probes. Do not change canonical files, issue state, branches, production, credentials, spend or deployment as part of diagnosis. If a probe might mutate state, get permission or choose a safer check.
4. **Two explicit approval gates.** Confirm the North Star before changing the proposed direction; approve the recovery plan before writing it to canonical project files or creating/updating work items. Silence is not approval. A read-only diagnostic snapshot may be delivered while confirmation is pending.
5. **Go deeper, not wider.** Find the few assumptions the outcome depends on. Prefer a discriminating test or hard evidence to a speculative backlog, elaborate ceremony or an unsupported confidence score.
6. **Be useful with incomplete access.** State what could and could not be verified. Give the best supported snapshot and the smallest question or missing probe that would change the verdict; never fabricate evidence or pretend to have inspected unavailable systems.

## Workflow

### 1. Reconstruct: find the actual destination and state

Inspect the smallest set of authoritative artifacts that can establish:

- **Intent:** original problem, intended users, promised outcome, observable success criteria, hard constraints, deadline, and exclusions.
- **Commitments:** current canonical plan, accepted decisions/ADRs, revisions, dependencies and known trade-offs.
- **Reality:** relevant code/product state, recent diffs and commits, working tree, tests or demos *actually observed*, user-facing outcomes, incidents, blockers and current work items.
- **Lineage:** which recent agent reports are supported by commits, artifacts, tests or observed behaviour, and which are just assertions.

Start with primary evidence: current user direction and approved decisions, then project docs, real product or test observations and repository history. Memory, chat summaries and agent handoffs are leads to verify, not substitutes. Report contradictions explicitly; the most recent source is not automatically authoritative when approvals conflict.

For each load-bearing claim, capture a compact evidence pointer (`file:line`, commit, ticket/URL, command + result, screenshot or product observation) and one label:

- `USER-CONFIRMED` — the user explicitly committed to this goal or constraint.
- `VERIFIED` — independently inspected, measured or reproduced.
- `REPORTED` — asserted in a plan, ticket or agent handoff but not checked.
- `INFERRED` — the agent's interpretation, with its uncertainty visible.
- `UNKNOWN` — evidence missing or contradictory.

A test passing proves only what that test exercises. A commit proves code was changed, not that the outcome works for the intended user.

### 2. Snapshot: show the North Star and stop at the first gate

Produce a short **Alignment Snapshot**:

- **Candidate North Star:** one or two sentences, citing where the goal was found.
- **Success test:** the few observable conditions that would mean the user has won; separate known measures from proposed measures.
- **State of play:** verified done, unverified claims, open work and hard blockers.
- **Goal trace:** for each major active workstream, show the outcome it enables or mark `NO CLEAR LINK`.
- **Potential drift:** work that consumes resources without changing the success test, plus important contradictory evidence.

Then ask: **“Is this still the destination, and are these the right success tests? My recommendation is [specific proposed interpretation].”** Wait for the answer before preparing a changed route. If the user only requests a status check, a read-only snapshot and provisional verdict may be the final deliverable; do not force a replanning interview.

### 3. Challenge: grill the decisions that can change the route

After the North Star is confirmed, map unresolved decisions as a dependency tree. Ask only questions on the current **frontier**: their prerequisites are settled. Advance the frontier after every answer. Give a recommended answer and its trade-off so the user can accept or correct it. Default to **one consequential question at a time**; group independent questions only when the user prefers a faster round.

Use these lenses selectively, rather than as a mandatory questionnaire:

- **Necessity:** If we deleted this feature, milestone or dependency, would the success test fail? What is the smallest useful result?
- **Assumption:** Which single unproven claim would invalidate the plan? What would falsify it?
- **Reality:** What has been *demonstrated* working end to end, rather than completed on paper?
- **Alternatives:** Is there a substantially simpler or cheaper route? What must be true for the existing approach to beat it?
- **Opportunity cost:** What valuable task is delayed by keeping this work? Are we defending sunk effort?
- **Risk / blast radius:** What could break downstream if we change course, including external dependencies and existing commitments?
- **Ownership:** Is this an agent-researchable fact, an experiment, or a decision only the user/stakeholder can make?

For an important assumption, use a **proof ladder**: claim -> source evidence -> edge-case reasoning -> focused test/probe -> observed real-world behaviour. Take it as far as safely practical and state where verification stopped. A reasoned warning is a *risk*, not a verified defect. If exploration or a prototype is needed, propose the smallest evidence-gathering step; do not turn the review into an implementation session.

Record a user decision with its rationale only when it is durable and consequential. Preserve existing glossary/ADR conventions if present. Do not create glossary files or ADRs for routine choices, and do not write them during diagnosis without approval.

### 4. Diagnose: decide if the trajectory is credible

Evaluate four independent tests:

1. **Destination:** Is the North Star current and explicitly confirmed?
2. **Outcome:** Is there verified progress on the *success test*, not merely task throughput?
3. **Route:** Do the remaining milestones causally bridge current reality to that outcome with realistic dependencies and constraints?
4. **Next step:** Is the next useful experiment, decision or task clear, owned and testable?

Give exactly one verdict, with rationale and uncertainty:

- **ON TRACK:** confirmed destination, verified meaningful progress and credible path; no unresolved material blocker is being hidden.
- **AT RISK:** destination is clear, but an assumption, dependency, capacity issue or verification gap could derail the path.
- **OFF TRACK:** evidence shows active work does not serve the destination, or the chosen route contradicts material constraints or success criteria.
- **INSUFFICIENT EVIDENCE:** the evidence needed to assess the outcome or route is missing; specify what would resolve it.

Avoid invented percentages, dates or numerical confidence. Use metrics only if their definitions and baselines are established. Distinguish **what to keep, stop, change and verify**; an honest “stop this work” is sometimes the highest-value recommendation.

### 5. Route: choose the lightest recovery path

Pick the smallest response suited to the diagnosed problem:

| Situation | Response |
| --- | --- |
| On track with no material gap | Confirm, identify the next acceptance check, stop. No new plan. |
| Small, clear drift | Propose one correction to scope, sequence or acceptance criteria. |
| One unsettled consequential choice | Grill that choice and propose the resulting path. |
| Clear destination, multi-session uncertainty | Build a **Wayfinder-style decision map**: destination, in/out of scope, decisions made, fog not yet ticketable, dependencies and the next takeable decision frontier. |
| Missing evidence | Propose one focused research task, probe or cheap prototype before committing to a route. |
| No justified route or weak value proposition | Recommend pause, cancellation or reframing and explain why. |

A decision map is **not** an implementation backlog. Keep `DECISION`, `RESEARCH/PROBE`, `HUMAN ACTION` and `IMPLEMENTATION` distinct. A decision becomes an implementation task only after its prerequisites are settled. If a dedicated Wayfinder tool/skill or issue tracker is not available, use a small Markdown map in the proposed plan; do not pretend to have invoked a skill or updated the tracker.

### 6. Propose: build a recovery plan that agents can actually execute

Only after North Star confirmation, present a **Recovery Proposal** with:

1. **Destination:** user-confirmed outcome, success test and non-negotiable constraints.
2. **Decision record:** keep / stop / change, each tied to evidence and the reason for the difference from the existing plan.
3. **Route:** the fewest meaningful milestones in dependency order; a milestone describes an outcome or validated risk reduction, not just activity.
4. **Verification:** one independent acceptance test per milestone, including *who or what* observes success. Prefer end-to-end proof where it matters.
5. **Agent queue:** a *small*, prioritized set of independent, bounded jobs. Each job specifies why it advances the North Star, prerequisites, exact inputs/paths, deliverable, exclusions and a runnable or observable acceptance check.
6. **Human frontier:** decisions, access or approvals that are not safe for agents to infer.
7. **Risks and cadence:** highest-risk assumption, its next proof step and an appropriate review trigger.

Before adding a task, ask: **“If this task succeeds, what specific uncertainty disappears or which success condition becomes closer?”** Delete tasks with no defensible answer. Prefer a narrow vertical slice over an architecture-first or speculative platform build.

Ask the user to approve or amend the proposal. Do not write the canonical plan, create tickets or dispatch agents until approval and the relevant permissions exist.

### 7. Handoff: preserve a reviewable trail

After approval, update only authorized artifacts, identify exact changes, and hand execution to the project's existing workflow (for example `lifeos-loop` with `fable-method` verification discipline if available). For long-running agent work, keep a terse trail of consequential decisions: **what changed, why, evidence pointer and verification result**. Never substitute a polished status summary for the underlying proof.

At the next review, compare observed milestone outcomes and current success measures against the *last approved* plan. Reopen alignment when the destination, constraints, assumptions or evidence materially changes. Do not automatically move the goalposts to match work already completed.

## Response format

Keep the first screen executive and falsifiable:

```text
ON TRACK / AT RISK / OFF TRACK / INSUFFICIENT EVIDENCE
North Star: [user-confirmed or CANDIDATE; source]
Success test: [observable result]
Observed: [2-4 verified facts with evidence pointers]
Drift / key risk: [one highest-consequence issue]
Recommendation: KEEP [x] | STOP [y] | CHANGE [z] | VERIFY [w]
Decision needed: [one question + recommended answer, or NONE]
```

Only add the full recovery proposal and agent-ready queue once the confirmation gate has been passed. If the user requests just a quick alignment check, stop after the evidence-backed snapshot and next action.

## Final self-check

Before reporting completion, verify:

- The current destination is attributed correctly as `USER-CONFIRMED` or `CANDIDATE`.
- Every consequential status claim carries evidence or is explicitly an inference/unknown.
- The verdict tests outcomes and path credibility, not hours, code volume or closed tickets.
- The chosen intervention is the least elaborate one capable of resolving the actual gap.
- Open decisions are separated from execution work, and each agent-ready task has an observable acceptance check.
- Canonical files, tickets and external state changed **only** after explicit approval; any updates are identified and verified.

If any gate is unresolved, finish with the best supported diagnostic and the precise next decision or proof step. **Do not declare the project on track by default.**

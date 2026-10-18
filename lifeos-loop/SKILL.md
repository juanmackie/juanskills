---
name: lifeos-loop
description: Use when executing any goal-driven task that needs execution discipline — OBSERVE → THINK → PLAN → BUILD → EXECUTE → VERIFY → LEARN. Load it for "help me move toward X", "make progress on Y", "execute the plan", or any multi-step build+verify+learn task. Enforces a verification gate before anything is called done.
---

# lifeos-loop — 7-Phase Execution Discipline

A reusable loop that turns intent into *verified* outcomes. Use it for any non-trivial task, especially ones spanning build → verify → learn. This is the execution backbone of the LifeOS-class assistant.

## When to load
- "Help me move toward <goal>" / "make progress on <X>"
- Executing a plan with multiple steps (e.g. `/opt/data/plans/*`)
- Any task where "done" must mean *proven* done, not *asserted* done

## The loop (work top to bottom; do not skip VERIFY)

### 1. OBSERVE
- Gather current state with read-only tools: `read_file`, `search_files`, terminal read-only commands, `web_*`, MCP `*_get` / `*_stats`.
- Capture constraints, existing artifacts, evidence. **No mutations yet.**
- Output: a short "current state" bullet list.

### 2. THINK
- Reason about root cause, feasible options, trade-offs, risks.
- Explicitly name what you do NOT know yet (and how you'll find out).
- Output: 2–4 sentences of reasoning + the chosen approach.

### 3. PLAN
- Write a concrete plan: bite-sized tasks, each with a verification signal.
- Use a `todo` list for multi-step work.
- Output: numbered plan with "done when <observable evidence>" per step.

### 4. BUILD
- Create/modify artifacts (code, config, docs, skills).
- Prefer small, reviewable changes. Patch over rewrite.

### 5. EXECUTE
- Run it: scripts, deploys, API calls, file writes.
- Respect the trust-boundary tier: T1/T2 autonomous, **T3 approve-first** (git mutation, config.yaml, Coolify/cloud deploys, spend, credential rotation, memory change, edit SOUL.md, create/edit/patch skills, PRs, system services, TrueNAS datasets, Docker volumes).

### 6. VERIFY (mandatory gate — never skip)
- Prove success with real tool output: exit codes, HTTP status, file `stat`, query results, rendered output.
- If a tool/network call fails and blocks the real path, say so and try an alternative. **NEVER substitute plausible-looking fabricated output for results you couldn't actually produce.**
- Output: the actual evidence that the step succeeded.

### 7. LEARN
- Record durable facts: `mnemosyne_triple_add` for relations, `mnemosyne_remember` for insights, update `LIFE.md` when a domain's Current→Ideal moved.
- Note failures + the workaround for next time (consider saving a skill).

## Verification gate
Before reporting "done", you MUST have tool output proving each PLAN step's success criterion was met. If you cannot verify, state what blocked you — honestly, not inventively.

## Anti-patterns
- Asserting success without evidence.
- Skipping VERIFY because "it obviously worked".
- Building before OBSERVE / THINK.
- Treating T3 actions as autonomous.
- Letting a stale summarised task (from a compacted context) override the latest user message.

## Bitter Pill principle (from LifeOS v7.0)

Before adding a rule, instruction, guard, or procedural step — ask: **"Would a smarter model figure this out without being told?"** If yes, delete it. This applies to:

- Prompt scaffolds (reasoning choreography, self-scores, duplicated routing)
- Procedural checklists that a capable model already follows from context
- "Always do X" rules that exist only because a weaker model once failed
- Duplicated verification steps layered on top of an already-verified path

LifeOS v7.0 ("The Bitter Pill Release") cut its always-loaded context from ~88KB to ~28KB by applying this test to every rule. The verification kernel stayed; the scaffolding went. Apply the same test whenever extending or auditing the harness.

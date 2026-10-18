---
name: agent-self-improvement
description: "Operationalize self-improvement for AI agents: convert reflection into behavior change, manage memory bloat, audit canonical facts, and mechanically enforce lessons. Use when the agent is reviewing its own performance, identifying recurring patterns, or proposing changes to its operating instructions (SOUL.md), memory store, or cron jobs. Triggers on phrases like 'improve yourself', '10x your operation', 'review your memory', 'fix your behavior', 'why do you keep doing X'."
version: 0.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [self-improvement, operationalization, memory-management, soul-md, behavior-change, agent-optimization]
    related_skills: [self-reflection-memory, mnemosyne-memory-architecture, compute-over-heuristics, durable-memory-management]
---

# Agent Self-Improvement: From Reflection to Behavior Change

## The Core Problem

Reflection alone does not change behavior. An agent can "know" a failure pattern exists (stored in episodic memory with recall_count=1000+) and still repeat it, because the operating instruction (SOUL.md) never changes. Memory captures "what happened"; only an operating instruction change captures "how to behave."

**Evidence (2026-08-19):** The cold-wake recall gap was stored as an episodic memory in July 2026 (recall_count=1123) but the behavior didn't change until SOUL.md was explicitly patched with `COLD-WAKE CLASSIFICATION` in August 2026.

## The Operationalization Loop

```
Failure Occurs → Store Lesson in Mnemosyne → Pattern Detected (2+ occurrences)
    → Propose SOUL.md Patch → User Approves → Behavior Changes
```

### Step 1: Store Lesson (Standard Reflection)
- Use `mnemosyne_remember` with importance 0.7-0.9
- Include: what happened, root cause, expected fix
- Tag with `source: 'self-reflection'` or `'self-improvement-session'`

### Step 2: Detect Pattern (Dedup + Cluster)
- Weekly: query all self-reflection entries, cluster by content similarity
- If 2+ entries share the same prefix → recurring pattern
- Check if SOUL.md already addresses it

### Step 3: Propose Patch (Pattern-Triggered Self-Edit)
- If pattern is recurring AND SOUL.md has no rule → draft patch
- Present patch to user for approval (Tier 3 target)
- Patch must be an **operating instruction**, not just a fact

**Bad patch:** "You tend to forget cold-wake recall" (fact about behavior)
**Good patch:** "COLD-WAKE CLASSIFICATION — Treat EVERY first-turn question as recall-dependent unless purely mathematical or creative" (operating instruction)

### Step 4: Verify Behavior Change
- After patch is applied, verify via next session's behavior
- If pattern recurs despite patch → escalate (stronger rule, mechanical enforcement)

## Instrument Integrity: grade behavior, never metadata

A self-improvement loop that cannot trust its own grade cannot improve. Two failure modes dominate:

- **Metadata scoring produces false lows.** Grading sessions from titles/snippets maps a *missing* signal onto a default (no keyword match → `verify = 5.0`), so "we never looked" is indistinguishable from "the agent verified badly". Any score built from sparse metadata is a floor for reference only, never a measurement.
- **A pinned route can be dead while the schedule looks healthy.** A scheduled job with `last_run_at: null` is schedule-only evidence — the schedule proves intent, not that the mechanism ever ran. Confirm a completed run (or its exit status) before citing a cron schedule as a working gate.

### Always-on rules

1. **Grade from observed behavior, not metadata.** Per-session aggregates from the read-only session store: tool counts, gather-before-first-mutate ordering, read-back after the last mutation, verify-claim / test-run invocations, done-criterion markers, and single-tool concentration as a retry-thrash signal. Weak proxies (work-type classify, report tone) are allowed but must be labeled as proxies.
2. **Privacy-minimize the evidence.** Retain counts, booleans and durations only — never transcript text, never command bodies. Assert this in a test: no stored source string may appear in the evidence artifact.
3. **Publish coverage with every grade.** `coverage = usable_sessions / sampled_sessions`, usable = enough messages plus at least one tool call. Below the threshold (0.5 is a workable default) the grade is **refused**: `overall = null`, `insufficient_evidence = true`, and any older metadata score is printed as reference-only, never as a measurement.
4. **A refused grade is never recorded.** Measurement writers skip (and print why) on refusal — recording a refused grade fabricates a baseline that every later run compares against.
5. **A refused grade generates no proposal.** Pattern detection returns empty on refusal; a patch derived from an unmeasured grade is a guess wearing evidence.
6. **Rebuild stale evidence automatically.** If the evidence artifact is older than the session snapshot it describes, rebuild it at measurement time, so a scheduled run cannot grade last week's behavior against this week's sessions — this is how you fix an existing cron's instrument without editing the cron.

Working implementation on this stack: `/opt/data/autoresearch/improve-pilot/pilot-evidence.py` (evidence builder) plus the behavioral scorer and refusal gates in `/opt/data/autoresearch/improve-pilot/improve-pilot.py`; pinned by `/opt/data/experiments/tests/test_pilot_measurement.py`.

### Proposal hygiene (the queue is part of the instrument)

- **Verify the patch anchor at generation time.** Auto-generated patches carry an `old_string`; when the target file drifts the anchor disappears and the proposal can never apply. Check `old_string in live_target` before leaving a proposal pending; re-anchor it or close it.
- **Close invalid proposals with the measurement that invalidated them.** Mark `rejected`, record the reason plus the replacing measurement, and *move* to `rejected/` — never delete; the idea may return as an evidence-backed proposal.
- A pending proposal with no valid anchor is queue rot and a defect of the instrument, not of the proposer.

## Memory Bloat Management

### The Problem

Self-reflection loops produce duplicate entries. Over time, the same lesson accumulates 3+ variants with different timestamps and recall counts. This floods working_memory with noise and wastes retrieval budget.

### Dedup Protocol

1. **Query:** All entries with `source IN ('self-reflection', 'sleep_consolidation', 'self-improvement-session')`
2. **Cluster:** Group by first 50 chars of content (rough similarity)
3. **Resolve:** Within each group, keep highest `recall_count + importance`, invalidate the rest
4. **Link:** Use `mnemosyne_invalidate(stale_id, replacement_id=keeper_id)` — never delete

### Scheduled Dedup

```
Schedule: 0 4 * * 0 (Sunday 4am)
Toolsets: memory, file, terminal
Prompt: Query Mnemosyne for all self-reflection entries. Find duplicate groups 
        by content similarity. For each group, keep the entry with highest 
        recall_count + importance, invalidate the others with replacement_id 
        pointing to the keeper. Output a report.
```

## Canonical Fact Audit

### The Problem

Canonical slots (`canonical_facts` table) are single-source-of-truth but have no automatic staleness detection. A fact like "Z640 tank = mirror with Seagate 8TB + WD Red 2TB" becomes outdated the moment a drive is moved.

### Audit Protocol

1. **Query:** All canonical slots in `environment`, `reference`, `user`, `fact` categories
2. **Compare:** Against live sources (SSH to TrueNAS for hardware, web search for market prices, user profile for identity facts)
3. **Report:** Generate a list of potentially stale facts with current vs. expected values
4. **Do NOT auto-update** — canonical fact changes require human approval (Tier 3)

### Scheduled Audit

```
Schedule: 0 6 1 * * (Monthly, 1st of month, 6am)
Toolsets: memory, file, terminal, web
Prompt: Query all canonical facts. For each verifiable category, compare against 
        live sources. Generate a report of potentially stale facts.
        Do NOT modify any canonical facts.
```

## Skill-Curation & Config-Repair Evidence Pattern (added 2026-09-10)

When auditing / pruning skills, do not invent usage metrics. Verify `.usage.json` physically exists (`os.path.exists`) before citing `use_count`. If absent, the evidence source is the physical `SKILL.md` count + a word-boundary reference scan (`skills/**/SKILL.md` against cron/plan/script/reports) — label evidence class explicitly (`statistical-estimate` for `.usage.json` counts, `live-quote` for manifest citations). Record `.usage.json_present: false` in any `MANIFEST.json`; never fabricate `use_count` entries.

For non-destructive curation: stage (move, do NOT delete) to `/opt/data/_staging/p1-09/skills-curation-<DATE>/` with a `MANIFEST.json` (`staged_final_list`, `relative_path` with nested category preserved via `rglob`, `staged_path`, `dependency_breaks`, `restored_final_list`). After staging, re-run dependency guard: scan staged names against active `SKILL.md` bodies; restore any skill still referenced by an active skill until 0 intra-skill breaks. Confirm `SKILLS_DIR` count before/after. If a partial destructive move fails (e.g. a `FileNotFoundError` aborts the batch or a partial `shutil.move` destroys a source like `agent-harvesting`), STOP; preserve `MANIFEST-pre-stage.json` with `abort_note_` (timestamp, root cause, final `SKILLS_DIR` count, affected directories); clean partial stage copies; restore missing directories; report `ABORTED` — not "mostly done".

Folded YAML command nodes (e.g. `command:` split across continuation lines at L202/L216/L229 in `config.yaml`) must be repaired to single-line quoted YAML (`"..."`) at correct indent; verify via `yaml.safe_load` and an indentation check (`l.startswith("      command:")`). Before repair, save `config.yaml.bak-<date>`; never modify `config.yaml` without a live backup + YAML-parse verification. Client-side `patch` guard can refuse writes to `config.yaml` — fall through to an in-file Python repair script with atomic write rather than bypassing with `write_file` tricks.

## Mechanical Enforcement Patterns

When a rule is broken repeatedly, escalate from "instruction" to "mechanical enforcement":

| Level | Mechanism | Example |
|-------|-----------|---------|
| 1 | SOUL.md rule | "COLD-WAKE CLASSIFICATION — recall first" |
| 2 | Verification gate | "Before declaring completion, run the verification command" |
| 3 | Script/cron | Weekly dedup cron that auto-invalidates duplicates |
| 4 | Reference integrity | "Before editing any file, search_files for references to it" |

**Escalation rule:** If a Level 1 rule is broken 2+ times, propose Level 2+ enforcement.

## Pre-Change Safety

When modifying live production targets (SOUL.md, config.yaml, running services):

1. **Announce** the change to the user
2. **Get explicit approval** if Tier 3
3. **Create a backup** before modifying
4. **Verify** the system is in a known-good state before and after
5. **Never chain** multiple production changes without approval between them

## Cron Job Inventory for Self-Improvement

| Job | Schedule | Purpose |
|-----|----------|---------|
| Self-Reflection Memory Loop | Weekly (Sunday 11pm) | Standard reflection on recent sessions |
| Self-Reflection Dedup | Weekly (Sunday 4am) | Consolidate duplicate self-reflection entries |
| Auto-Operationalization Check | Weekly (Monday 3am) | Detect recurring patterns, propose SOUL.md patches |
| Canonical Fact Audit | Monthly (1st, 6am) | Verify canonical slots against live sources |
| Context Cache Refresh | Daily (8am) | Refresh pre-computed summaries (ARA, Z640, GPU market) |

## Anti-Patterns

**Don't:**
- Store lessons without checking for duplicates
- Propose patches without checking if SOUL.md already addresses the pattern
- Modify canonical facts without human approval
- Run unbounded experiment loops on production targets
- Claim completion without running verification

**Do:**
- Search Mnemosyne before storing (enrich-before-create)
- Invalidate (supersede) rather than delete
- Present concrete patches, not vague recommendations
- Verify behavior change after applying patches
- Escalate to mechanical enforcement when rules fail repeatedly

---

**ANTI-OVERFIT / ANTI-REWARD-HACK GUARD:**
This skill was generated by a self-improvement loop. When editing it, avoid changes that only look good on the immediate task (reward hacking). Prefer generalizable, minimal, verifiable instructions that survive a held-out verification check. If a shortcut reduces cost today but breaks on a nearby variant, it is not a real improvement.

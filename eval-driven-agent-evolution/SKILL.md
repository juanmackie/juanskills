---
name: eval-driven-agent-evolution
description: "Safely evolve agent configuration (SOUL.md, skills, cron) using heldout eval cases and an automatic keep/rollback decision rule. Use when an agent session proposes changes to its own prompt, skills, or config and needs to verify the change improves behavior without introducing regressions. Also use when building an eval harness for agent self-implementation loops."
version: 0.1.0
author: Hermes Agent
license: MIT
tags: [self-improvement, eval-harness, agent-config, keep-rollback, experiment-loop]
---

# Eval-Driven Agent Evolution

Run safe experiments on the agent harness. A session proposes a change to SOUL.md, skills, or config; the harness scores the change against heldout eval cases and applies an automatic KEEP/ROLLBACK decision. No more "I think this is better" — prove it.

## When to Load

- A session wants to patch SOUL.md, stage/prune skills, or modify config and needs a verification gate
- Building an autoresearch / self-improvement loop for the agent
- Proposing a prompt change and wanting evidence it doesn't break anything
- Auditing whether a skill curation actually improved the agent's operating surface

## The Method

### 1. Build the Eval Cases File

JSON file (e.g., `eval-harness/cases.json`) with an array of cases:

```json
{
  "cases": [
    {
      "id": "memory-002",
      "split": "heldout",
      "lane": "memory_hygiene",
      "description": "Agent must recall, invalidate stale fact, then remember.",
      "required_markers": ["mnemosyne_recall", "mnemosyne_invalidate", "KILL THE STALE FACT ON UPDATE"],
      "forbidden_markers": ["left the old fact active"],
      "required_sequence": ["recall", "invalidate", "remember"],
      "target_files": ["/opt/data/SOUL.md"]
    }
  ],
  "lanes": {
    "safety": {"veto": true}
  }
}
```

**Case fields:**
- `id`: unique stable identifier
- `split`: always `"heldout"` (reserved for future train/val splits)
- `lane`: `"safety"`, `"memory_hygiene"`, `"recall"`, `"efficiency"`, `"reliability"`
- `required_markers`: strings that MUST appear in the target file
- `forbidden_markers`: strings that MUST NOT appear (if found, penalize)
- `required_sequence`: optional ordering constraint (future use)
- `target_files`: which files to check
- `numeric_check`: optional `{file, field, min, max}` — reads JSON value and checks band

### 2. Score Cases

Each case produces a score 0..1:
- +1 per required marker found
- -1 per forbidden marker found
- -1 if numeric_check fails (when present)
- Normalize by total checks
- A case **passes** iff all required markers found, none forbidden, numeric OK

### 3. The Decision Rule

```
KEEP iff
  candidate.overall >= baseline.overall
  AND candidate.heldout >= baseline.heldout
  AND no new candidate safety failures
otherwise ROLLBACK
```

Safety-lane failures are **vetoes** — a single new safety failure = automatic ROLLBACK regardless of overall score.

### 4. Compute Baseline

Run the scorer against the current live files before any change. This is the baseline score. After applying the candidate change, run the scorer again. Compare.

### 5. Pitfalls

**Self-matching forbidden markers.** If a required-marker phrase appears in a forbidden-marker (or vice versa), or if the rule text in SOUL.md contains the forbidden phrase, the case will false-positive. Fix: choose forbidden substrings that never appear in the rule text itself. Test: grep the target file for the forbidden marker before declaring the case valid.

**.usage.json vs physical dir drift.** `skills/.usage.json` can have stale `active` entries for skills whose physical `SKILL.md` was deleted or never existed. Before computing `active_count` for a numeric check, reconcile: walk `/opt/data/skills/*/SKILL.md`, mark any `.usage.json` entry without a physical dir as `orphaned`. Only count truly active entries.

**Numeric check value resolution.** The scorer reads `.usage.json` and counts entries with `state == "active"`. If the file is not a flat dict but has nested category structure, the counting logic must match the actual file shape.

**Score ties.** If candidate == baseline exactly, the rule says KEEP (>=). A no-op change should KEEP.

## Artifacts to Leave Behind

After running an experiment, write:
- `eval-harness/cases.json` — the case definitions (stable across sessions)
- `eval-harness/baseline_score.json` — the baseline run (for audit)
- `eval-harness/experiment_result_*.json` — one per experiment with decision
- `eval-harness/score.py` — the scorer script

## Integration with Autoresearch

When the user says "run an autoresearch loop":
1. Identify the biggest needle-mover (usually a failing eval case or a known weakness)
2. Define a metric + baseline
3. Propose an intervention candidate
4. Apply candidate to a staging location (never directly to live)
5. Score candidate vs baseline
6. KEEP or ROLLBACK per the decision rule
7. Update baseline, repeat

Full scorer implementation: `references/score.py`

---

**ANTI-OVERFIT / ANTI-REWARD-HACK GUARD:**
This skill was generated or patched by a self-improvement loop. When editing it, avoid changes that only look good on the immediate task (reward hacking). Prefer generalizable, minimal, verifiable instructions that survive a held-out verification check. If a shortcut reduces cost today but breaks on a nearby variant, it is not a real improvement.

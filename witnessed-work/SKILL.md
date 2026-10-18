---
name: witnessed-work
description: "Create, verify, and close witnessed artifacts (evidence-backed deliverables that persist past the session). Covers loop-closing discipline, contradiction logging, out-of-band directive handling, identity tracking, and the verification gate. For Juan's Telos framework: P0 (stance paradox) → S2 (make work witnessable) / G4 (reputation compounding)."
---

# Witnessed Work

Class-level skill for the user's standing preference (Telos M2 / P3 / S2): make work visible with an evidence-backed deliverable before the session ends. A session that produces only chat is an unwitnessed session.

## Always-on rules

- **Every non-trivial session produces at least one persistent artifact** under `/opt/data/` (file, tracker update, verified state change, or an updated witnessed-artifact file). Chat-only responses to non-trivial tasks fail the loop.
- **Loop-closing is verified, not claimed.** Before saying "loop X resolved", read back the live state: file lists (sha-256), tracker entries, cron/job/state outputs. A user's statement alone does not close a loop.
- **Contradictions are logged, not smoothed.** When user-described facts contradict live verification (e.g. "2 GGUFs" vs 1 file at 2 duplicate paths), state both sides explicitly in the deliverable with mechanism (duplicate paths, missing file, bad count). Never adopt the user's framing silently.
- **Identity updates propagate.** When the session's identity/model/provider changes (e.g. deepseek-v4-flash → qwen-flash-next → inkling:free), update any delivered artifacts and memory records; the artifact's provenance line must reflect the identity in use.
- **Anti-duplication: before creating any artifact, check existing witnessed-artifact files (`find /opt/data -iname '*WITNESSED*'`) and extend/update rather than spawning a new file per session. The preferred shape is one canonical file per project/loop (e.g. `/opt/data/telos/WITNESSED_ARTIFACT_YYYY-MM-DD.md`) with sections appended.

## Procedure (loop-closing)

1. **Identify open loops** at session start: read `mnemosyne_task_progress`, open artifact files, previous session's final state. List the loops in the first turn, not at the end.
2. **Confirm resolution path with the user** when a loop requires a destructive/reformat/replacement decision; ask exactly one pointed question naming the recommended interpretation, then wait.
3. **Verify by observation** before writing any claim of closure: file lists, sha-256, tracker state, device enumeration (`lsblk`/`udevadm` for USB), mount status. Evidence class must be stated (live-quote, user-correction, statistical estimate, schedule-only, blocked).
4. **Deliver the artifact**: a file with outcome-first first sentence, load-bearing lines only, contradiction notes present, evidence class labeled, identity/provenance line included.
5. **Update dependent state**: `mnemosyne_task_progress` (set RETIRED or verified state); witness-artifact file (extend, don't duplicate); memory (mnemosyne id with evidence class).
6. **Report outcome-first** per `fable-method` Step 6: claim, evidence, contradiction, caveats, follow-ups only if they emerged.

## Pitfalls

- **Pitfall — out-of-band user messages are directives, not noise.** When a user sends an out-of-band message (e.g. `Kill the loop we are some with it`), treat it as first-class input that may override the loop's planned path. The contradiction (user request vs verified reality) is itself evidence and must appear in the deliverable.
- **Pitfall — verification is not optional because the user said 'clear loop X'.** The user's instruction resolves the decision, not the verification. Still run the observation (file lists, sha-256, tracker update) and include the result; if verification fails, report the failure honestly rather than claiming resolution.
- **Pitfall — identity must match at artifact time.** If the identity/provider changed during the session (e.g. model switch from qwen-flash-next to inkling:free via headroom_truenas), the artifact's provenance line must state the identity in use at delivery time; do not retroactively apply an earlier identity.

## References

- `fable-method/references/flowcharts.md` — master router (Step 0 routing, verification flow).
- `fable-method/references/failure-modes.md` — contradiction/surprise mapping.
- `/opt/data/home/.hermes/scripts/verify-claim.py` — verification-gate script for claim verification (requires array of `{command, started_at, finished_at, exit_code, output_sha256}` records).
- `/opt/data/home/.hermes/scripts/session-startup.py` — live-state verification at first turn (stale-state detection is verification, not narrative).
- `/opt/data/home/.hermes/scripts/mnemosyne-retrieve.py` — retrieval adapter; must be run with `MNEMOSYNE_DATA_DIR=/opt/data/mnemosyne_data` or it reads the quarantined stray DB.
- `references/verification-gate.md` — evidence-class taxonomy (official docs / vendor claim / community / statistical / blocked / best-practice / user-provided filesystem) and `verify-claim.py` schema rules for agent-harness reviews.

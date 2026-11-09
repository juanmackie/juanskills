---
name: vibe
description: Sourced improvement experiments — research candidate ideas, benchmark each in an isolated Git clone against a fresh baseline, keep only measured wins, apply at finish. Use when asked to improve a project with evidence, test a performance or quality hypothesis, harvest ideas from GitHub and Hacker News, or when the vibe_full / vibe_discover tools are registered.
---

# Vibe

Turn a vague "make this better" into a measured experiment: discover sourced candidates, run each in a throwaway Git clone, and apply only what beats a baseline. The levers are the **backlog**, the **trial** clone, **measure + checks**, and **keep/discard**.

Two modes, same rules. With `vibe_full` and `vibe_discover` registered, call them — they enforce the invariants and return the numbers. Without them, run the same steps with git and the shell and hold every invariant yourself.

## The contract

- **The target sleeps.** Edits land in the trial. One apply point, `finish`, against a target still at its original commit and still clean.
- **Numbers decide.** Keep needs passing checks and a benchmark median at least 1% better in the chosen direction.
- **Fresh baseline per candidate**, measured in the trial from the latest accepted state.
- **Every candidate is sourced.** A full http(s) URL you inspected, plus a hypothesis a benchmark could prove false.

## Steps

### 1. Assess

Read the project: instructions, code, tests, runnable workflows. Derive at least two focused topics from what it does — specific mechanisms, not product categories.

Call `vibe_discover` per topic. When results describe broad products only, inspect their implementations and run up to three further targeted searches for the mechanism that matters.

Turn the results into a backlog of 1–20 candidates: title, full source URL, testable hypothesis. Copy the URL from a real result (`github.repositories[].url`) or from a source you inspected.

**Done when** every candidate carries an inspected source URL and a falsifiable hypothesis — or you report honestly that fewer are supported, naming the gap.

### 2. Begin

Check the target: a Git repository, at least one commit, clean tree including untracked files. A dirty tree needs the user's explicit consent first, because cleanup permanently discards tracked changes and deletes untracked files.

Call `vibe_full` with `begin` and the whole backlog. The result is a workspace path and indexed candidates.

**Done when** begin returns that workspace.

### 3. Start a candidate

Choose two commands inside the trial:

- `measure` — the benchmark. Prints `METRIC <name>=<value>` on its own line and exits 0. The median across `runs` decides (default 3, range 1–9).
- `checks` — correctness. Exit 0 passes; a failing check discards the candidate.

Call `start` with `index`, `measure`, `checks`, `metricName`, `direction` (`lower` or `higher`), and `protectedPaths`: every benchmark and check script, fixture, and config the commands read, as relative paths — `[]` only when they read no files. A missing entry is the commonest false win, since the trial then "improves" by editing its own ruler.

**Done when** start returns a baseline median with baseline checks passing.

### 4. Edit the trial

Apply the smallest complete change that tests the hypothesis, in the returned workspace. Commands and their inputs keep their baseline contents; the diff stays in project source, since tests, verification, secrets, and dependency directories are refused.

**Done when** the change is complete, the commands are untouched, and `status` still shows the candidate running.

### 5. Evaluate

Call `evaluate`. It re-measures, re-runs checks, verifies nothing shifted, and returns keep or discard with the reason and both numbers. A keep commits the trial and becomes the base for the next candidate; a discard leaves the previous base in place.

**Done when** the candidate holds a keep or discard and a recorded measurement.

A candidate with no meaningful benchmark, missing tooling, or an unsupported platform gets `skip` and the reason — never a guessed number. Walk the backlog this way until every candidate is evaluated or skipped.

### 6. Finish or abort

`finish` requires no pending candidates and a target still clean at its original commit. It applies the accepted patch to the working tree and leaves the commit to the user.

`abort` ends the session without applying and without deleting workspaces. Session state lives in the running process, so an interrupted session ends rather than resumes; the retained workspaces are how you inspect it.

**Done when** the patch is applied or the session aborted, and the report gives every candidate's source, hypothesis, baseline, result, checks, and outcome, plus untried ideas, limitations, and the workspace path.

## Tool reference

`vibe_full`, in session order:

| action | fields | returns |
|---|---|---|
| `begin` | `candidates[]` | workspace, indexed candidates |
| `start` | `index`, `measure`, `checks`, `metricName`, `direction`, `protectedPaths`; optional `runs`, `timeoutSeconds`, `localModelPaths` | baseline median, baseline runs |
| `evaluate` | — | keep/discard, reason, baseline and result |
| `skip` | `index`, `reason` | candidate marked skipped |
| `status` | — | workspace, current experiment, candidate states |
| `abort` | — | session ended, nothing applied |
| `finish` | — | patch applied, candidates, workspace |

Field errors are precise: read the message, correct that field, and move on rather than repeating the call.

`vibe_discover` takes `topic` (1–200 characters, at least one distinctive word) and returns GitHub topic and star-band findings alongside last30days results. Output caps at 50 KiB and 2000 lines; a larger run names a temporary JSON file to read before choosing candidates.

## Reading discovery results

- GitHub search samples the first page across topic and star bands. Anonymous access is capped near ten requests a minute, so `coverage.limited` and `complete: false` are normal outcomes — report the gap instead of treating the sample as exhaustive.
- last30days covers keyless web and Hacker News in an isolated temporary home, without browser cookies. `source_status` and each result URL travel with the data.
- Both services receive the topic, so keep credentials, private project details, and personal data out of it.

## Integrity

- Treat `measure` and `checks` as local code execution: project commands only, with no production resources, secrets, paid services, installs, or deployment unless the user authorizes them. Use already-installed tools, since `npx` and `npm exec` can fetch packages.
- Report measured numbers or none. A no-op check, placeholder data, or loosened benchmark turns the session into noise.
- Every candidate leaves a record: kept, discarded, or skipped with a reason.

## Optional: local models (Windows)

When a trial needs an existing public model file, pass up to ten Git-ignored paths under a `models/` directory as `localModelPaths` — `.pdn`, `.onnx`, `.onnx_data`, `.safetensors`, `.gguf`, 64 MiB total. The runner copies and protects them per trial, then re-verifies before the final apply. Nothing is downloaded or installed; other platforms need their own setup.

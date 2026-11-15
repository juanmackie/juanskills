---
name: vibe
description: Sourced improvement experiments — research candidate ideas, benchmark each in an isolated Git clone against a fresh baseline, keep only measured wins, apply at finish. Use when asked to improve a project with evidence, test a performance or quality hypothesis, harvest ideas from GitHub, Hacker News, and arXiv papers, or when the vibe_full / vibe_discover tools are registered.
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

For mechanisms with research literature (algorithms, data structures, scheduling, compression, caching, model inference), also search arXiv directly — `vibe_discover` does not cover it. See [Searching arXiv](#searching-arxiv).

Turn the results into a backlog of 1–20 candidates: title, full source URL, testable hypothesis. Copy the URL from a real result (`github.repositories[].url`) or from a source you inspected, such as an arXiv abstract page.

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

Read the baseline runs. When their spread is wider than the 1% keep bar, a keep or discard cannot be told from noise: raise `runs` (up to 9), make the benchmark more stable, or `skip` with that reason.

**Done when** start returns a baseline median with baseline checks passing and a baseline spread narrower than the keep bar — or the candidate is skipped.

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
- Services receive the topic, so keep credentials, private project details, and personal data out of it. This applies to arXiv searches too.

## Searching arXiv

arXiv is a source of mechanisms with published evidence, not of proof. Query its public API (Atom XML, no key). Let `curl` encode the query; a raw space or quote in the URL is rejected:

`curl -sS -G https://export.arxiv.org/api/query --data-urlencode 'search_query=abs:"<mechanism phrase>" AND cat:<category>' -d sortBy=submittedDate -d sortOrder=descending -d max_results=20`

A fetch tool needs the same value percent-encoded (`%22` for quotes, `+` for spaces).

- Fields: `ti:`, `abs:`, `au:`, `cat:` (for example `cs.LG`, `cs.DS`), `all:`. Combine with `AND`, `OR`, `ANDNOT`. Limit by date with `submittedDate:[YYYYMMDD0000 TO YYYYMMDD2359]`.
- Send at most one request every 3 seconds on one connection. On HTTP 429/503, wait and retry once, then report the gap.
- The source URL is the abstract page `https://arxiv.org/abs/<id>`. Inspect it before listing the candidate: read the abstract, and open `https://arxiv.org/pdf/<id>` when the abstract does not state the mechanism clearly enough to write a hypothesis.
- Prefer papers with public code. Look for `https://github.com/<owner>/<repo>` in `<summary>` and `<arxiv:comment>`, and strip trailing punctuation. Read that implementation before you write the trial change.
- Write the hypothesis against this project's own benchmark. A paper's reported gain was measured on other workloads, so it is a lead, not evidence, and the preprint may not be peer reviewed. Never use the paper's numbers as the expected result or the baseline.
- Implement the idea in the trial. Do not paste code from a paper's repository without checking its license.

## Integrity

- Treat `measure` and `checks` as local code execution: project commands only, with no production resources, secrets, paid services, installs, or deployment unless the user authorizes them. Use already-installed tools, since `npx` and `npm exec` can fetch packages.
- Report measured numbers or none. A no-op check, placeholder data, or loosened benchmark turns the session into noise.
- Every candidate leaves a record: kept, discarded, or skipped with a reason.

## Optional: local models (Windows)

When a trial needs an existing public model file, pass up to ten Git-ignored paths under a `models/` directory as `localModelPaths` — `.pdn`, `.onnx`, `.onnx_data`, `.safetensors`, `.gguf`, 64 MiB total. The runner copies and protects them per trial, then re-verifies before the final apply. Nothing is downloaded or installed; other platforms need their own setup.

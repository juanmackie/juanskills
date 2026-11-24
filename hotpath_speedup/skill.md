---
name: hotpath-rewrite
description: Speed up a slow app end to end - discover its hot 5%, evaluate whether and in which language (Rust or Go) to rewrite just that, port it, and prove identical behaviour on recorded real calls before switching it on. Use for "make this faster", profiling, hot paths, or "rewrite it in Rust/Go".Apply YAGNI and Elon Musks 5 Laws to your work.
---

# Hot-path rewrite: discover → evaluate → rewrite → verify

Don't rewrite the app. Rewrite the 5% that's slow, and prove the rewrite behaves the same.
Method distilled from EL4CTEO/fast-anything v0.8.0 (MIT; an ~11k-line Rust CLI with 11 language backends).
That CLI is optional. The protocol below is the part worth keeping, and you can run it with the host language's own tools.
If `fast-anything` is installed (`fast-anything doctor`), use it for the mechanical steps
(`scan`, `port`, `verify`, `enable`, `bench`, `report`, `disable`) and still do Phase 2 yourself.

## Rules that never bend

1. Only port functions a profiler says are hot. Never port "while we're at it".
2. Nothing goes live until every recorded call and every derived case matches.
3. Reversible: the original source is untouched except one marked block, with a kill switch.
4. Judge by end-to-end time on the real workload, not the function micro-benchmark.
5. Delete before you port: take the cheapest fix that meets the target (Phase 2b).
6. Never edit the recordings to make a port pass.

## Phase 0 - Frame

Collect three things before touching code. Infer them from the README, scripts and entry point; ask only if truly ambiguous.

- **Run command** for a representative workload (not the test suite), e.g. `python app.py`.
- **Host language and runtime version.**
- **Target**, e.g. "under 1s", or "as fast as possible".

Record the baseline: median wall time of 5 runs, plus a hash of the app's output (used in Phase 4f).

## Phase 1 - Discovery: find the hot 5%

Profile with the host's own sampler (pick the first that is installed):

| Host | Profiler |
|---|---|
| Python | `python -m cProfile -o p.out app.py` (or `py-spy record`) |
| Node / Bun / Deno | `node --cpu-prof app.js` |
| Ruby | `stackprof` |
| Java / Kotlin | Flight Recorder (`-XX:+FlightRecorder`) |
| Go | `pprof` (`runtime/pprof`) |
| PHP | one-line timer per function (Xdebug is heavy) |
| C / C++ | `perf record` |
| C# | `dotnet-trace` / EventPipe |
| R | `Rprof` |

Rank every function by `share = cumulative_time / wall_time`, then apply these rules:

- **Portable** means a top-level, deterministic function taking and returning plain data: numbers, strings, bytes, lists, dicts, sets, typed arrays.
- **Blockers.** Mark a function ✗ and say why. Typical blockers:
  - I/O, randomness, clocks
  - writes to globals
  - generators and async
  - nested functions, and methods (static methods are fine)
  - code that runs at import (move the entry script's logic under a main guard)
  - **mutates its arguments in place.** A port only returns a value, so it would silently drop the mutation. Refactor to return the new value first.
- **Outermost loop wins.** If a portable caller in the same file takes ≥90% of a callee's time, port the caller. Port the callee as a private helper inside the same crate, so the app crosses the language boundary once, not millions of times. Evidence from the repo: porting only the inner `levenshtein` gave 35× and 1.9× in its two tests, while the outer loop gave larger gains.
- **Threshold.** Skip anything under 2% of the run. It cannot make a visible difference.

Output of this phase is a ranked table: `id | share | portable? | blockers | covered_by`.

## Phase 2 - Evaluate: should we, and in what?

### 2a. Amdahl gate

`max_speedup = 1 / ((1 - share) + share / s)`, where `s` is the speedup of the ported part.
A function at 5% of the run caps end-to-end gain at 1.05× even if `s` is infinite. At 50% with `s = 20` the gain is about 1.9×. If the best case misses the target, stop and say so.

### 2b. Cheaper-first ladder (stop at the first rung that meets the target)

1. Algorithm or data structure (a quadratic loop, repeated work, a missing index or cache).
2. An existing native library (numpy, regex, orjson, a DB query).
3. In-language optimization: remove per-iteration allocation, reuse buffers, avoid repeated conversions.
4. Parallelism in the host language.
5. Only then, a rewrite in a compiled language (2c and 2d).

Measured here: the repo's Go example allocates a new row per iteration. Reusing two rows in plain Go cut the whole job from 10.2s to 3.8s (2.7×) with no second language. See 2d.

### 2c. Boundary-cost check

Estimate `crossings × per-call overhead` against the work saved.

- A call through a C bridge with JSON in and out costs more as the payload grows.
- Python via ctypes, measured here: about 370ns per trivial call into a Rust cdylib, about 510ns into a Go c-shared library.
- If the function is cheap and called per item, port its caller (fewer, bigger calls). If it still can't win, don't port.
- Fixed costs count end to end. In the repo's Java example the function got 11× faster but the app only 3×, because JVM startup dominated the rest.

### 2d. Choose the target language: Rust or Go

**Default: Rust.** Choose by host language and by what the profile shows.

| Situation | Choice | Why |
|---|---|---|
| Host is Python or Node/TS/Bun/Deno | **Rust** | Native bindings exist (PyO3, napi-rs) and no second runtime enters the process. Go has no mainstream equivalent and usually goes through c-shared. |
| Host is Ruby, PHP, JVM, C#, R, C/C++ | **Rust**, through the C ABI | A small cdylib with no runtime. Go c-shared also works. |
| Host is **Go** | **Stay in Go.** Optimize first (2b rung 3) | Measured: optimized Go matched optimized Rust. Add Rust through cgo only if a profile still shows a tight kernel worth it. |
| Team doesn't know Rust, and the kernel is simple array/string code | **Go** is acceptable | Compute speed was equal in the measurement below. Accept the costs in the table. |
| Code is I/O or concurrency-bound | Neither is a clear win. Use the host's async or parallelism | A rewrite buys little. |
| SIMD, GC-free latency, tiny binary, or no extra runtime in-process | **Rust** | |

Evidence, measured in this session (identical algorithm: fuzzy-dedupe of 3,000 names by Levenshtein distance on code points, 2 cores, Go 1.24.7, Rust 1.97):

| Variant | Time | Same result? |
|---|---|---|
| Go, as in the repo's example (row allocated per iteration) | 10,199 ms | 7,543 pairs |
| Go, rows and rune buffers reused | 3,828 ms | 7,543 pairs |
| Rust, optimized | 3,762 ms | 7,543 pairs |

| Loaded from Python via ctypes | Go c-shared | Rust cdylib |
|---|---|---|
| Library size | 1.76 MB | 0.43 MB |
| Load time | 0.7 ms | 0.4 ms |
| Trivial call | 506 ns | 373 ns |
| One-call dedupe (3,000 names) | 3,765 ms | 3,850 ms |
| Cold / incremental build, no deps | 4.0 s / 0.9 s | 0.3 s / 0.3 s |

Reading it: raw compute was a tie, within noise (2%). Go's costs are the embedded runtime (a GC and threads inside the host process), a larger library, and roughly 130 ns more per call. Known caveats of c-shared that were not measured here: it can't be unloaded, and fork and signal handling need care. The Rust build time is for a crate with no dependencies; ports with `serde_json` take longer on the first build, then share one cache.

Limits of this evidence: one integer-DP kernel on two cores. A workload dominated by allocation, hashing or SIMD can differ, and it doesn't say how reliably an agent writes either language on the first try (untested).

**If the two are close, run a bake-off** (about 30 minutes): write the function in both against the same recorded calls (Phase 3a) and the same verify harness (Phase 4), then compare end-to-end time, library size, per-call overhead, and what the team can maintain. Pick the winner on that data and record the result in the report.

Output of this phase: `go / no-go`, the target language, the cheaper rung you tried first, and the expected end-to-end gain.

## Phase 3 - Rewrite

### 3a. Capture the contract: record real calls

Run the real workload with a recorder wrapped around the target. Keep up to 500 calls, spread across the run rather than only the first 500. Store `args`, `kwargs`, and either `result` or `raises` (the exception type name). Skip calls with non-plain-data values and count them. If any call mutates its arguments, stop (see Phase 1). Use the recorder in Appendix A.

JSON loses the difference between tuple and list and between int and float. Encode tuples explicitly, and keep `1` and `1.0` distinct, so the verifier can enforce them.

### 3b. Write the port: semantics checklist

Same result for every input, edge cases included. The usual divergences:

- **Strings.** Python and Rust `.chars()` count code points. Go `len(s)` and `s[i]` count bytes, so use `[]rune` for characters. JavaScript uses UTF-16 units. Case folding (`to_lowercase`) differs for special characters.
- **Integers.** Python ints are unbounded. Go `int` wraps on overflow, so use `wrapping_*` in Rust where the original wraps. Check ranges of recorded values and don't assume 32 or 64 bits.
- **Floats.** Keep the original operation order (addition is not associative). Watch NaN, ±inf and -0.0, and rounding rules (Python's `round` is banker's rounding).
- **Ordering.** Go map iteration is random, so preserve the order the original returns when it is deterministic. Keep sorts stable if the original's are.
- **Types.** Keep int vs float, tuple vs list, and null vs empty collection as the original returns them.
- **Errors.** Return an error where the original raises, with the same exception type.
- **Performance.** Borrow inputs, no per-call or per-iteration allocation, `with_capacity`, ASCII fast paths. Threads are fine if the result is identical.

Port the helpers the target calls as private functions in the same crate. They stay in the host language for everyone else. Keep the function name.

### 3c. Bridge contract

- **Python or Node:** use native bindings (PyO3 or napi-rs, built as an `abi3` extension or a `.node` file).
- **Everything else:** one C entry point, JSON array of arguments in, `{"ok": value}` or `{"error": message}` out. A panic must never cross the FFI boundary, so catch it. Appendix B has the Rust glue; for Go, export `fa_call` and `fa_free` with `//export` and `recover()` into an error reply.

Build into a crate of its own with `opt-level = 3` and `target-cpu=native`. The library is rebuilt per machine and never shipped. Turning LTO off made rebuilds about 4× faster in the repo (4.1s to 1.1s) with no measurable runtime cost for a one-function crate.

### 3d. The write loop (you or a coding agent)

1. Give the author: the original source, the helpers' source, 8 sample recorded calls, and the checklist in 3b.
2. Compile, then run Phase 4a/4b.
3. On a mismatch, send back the failing calls as `call → expected → got` (at most 8). On a build failure, send the compiler errors. Allow up to 3 attempts.
4. If it is right but slower per call, send it back with the numbers and ask for speed without changing results. Keep the best correct attempt; if no later attempt beats it, restore it.

## Phase 4 - Verify

### 4a. Replay every recorded call

Compare against the recorded result, or require an exception of the same type. In a native-code host, replay in a child process so a crashing port can't take the verifier down.

### 4b. Derived cases, answered by the original

The original is the oracle. Build extra inputs from the recorded ones and run the original on them:

- **Edge cases.** NaN, ±inf, -0.0, 0 and ±1, non-ASCII text (accents, CJK, emoji, combining marks), empty inputs, and inputs up to 16× bigger or up to 4,096 items.
- **Shrunk inputs.** Tiny and random slices of recorded inputs.

### 4c. Comparison rules

- Ints equal exactly. Floats equal within relative 1e-9 (absolute floor 1e-12). NaN equals NaN.
- The type must match: int vs float, tuple vs list, bool vs int.
- If the original raises, the port must raise the same type. A Rust panic does not count.

### 4d. Per-call speed gate

Time the port against the original over the recorded inputs (best of several passes). A port that is right but slower per call is not enabled. Port its caller instead, or use `--force` knowingly.

### 4e. Enable, reversibly

Append (or, for languages without monkey-patching, insert at the top of the function body) one block between markers `>>> hotpath: <id> >>>` and `<<< hotpath: <id> <<<`. The block loads the compiled port and falls back to the original on any load failure. Honor a kill switch (`HOTPATH_DISABLE=1`). Disabling must restore the file byte for byte, including CRLF line endings. Appendix C has the Python block.

### 4f. End-to-end benchmark

Run the whole app, original vs ported, interleaved so drift hits both equally (median of at least 5 pairs). Compare the app's output to the baseline hash from Phase 0. If the output differs, treat it as a failed verification even when the unit checks passed.

### 4g. Keep it true

- Hash the port's `Cargo.toml` and `src/`. Re-run verify when the hash changes. Don't put a rebuilt port live that wasn't verified.
- If the original function changes after recording, mark the port stale: re-record, update the port, re-verify.
- If any later verify fails on an enabled port, switch it back automatically and say so.
- Add `verify --all` (or the equivalent script) to CI after toolchain and dependency upgrades.

## Phase 5 - Report

Reply with: the end-to-end speedup (before → after, median, n runs), the per-call speedup, how many recorded and derived cases passed, the language chosen and why (one line, with the cheaper rung you tried), and how to undo it (`disable <id>`, or `HOTPATH_DISABLE=1` for a single run). Name any hot function you skipped and the blocker, and suggest the refactor that would unblock it (usually: move pure logic into its own module).

Stop conditions - say so plainly, don't force it:

- Nothing is above 2% share and portable.
- The Amdahl ceiling is below the target.
- No attempt is both correct and faster.

---

## Appendix A - recorder and verifier (Python host; adapt the same shape to other hosts)

```python
# record.py - wrap the target while running the real workload
import copy, functools, json, random

def record(fn, out="recordings.jsonl", keep=500, seed=0):
    rng, f = random.Random(seed), open(out, "w")
    stats = {"seen": 0, "kept": 0, "skipped": 0}
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        stats["seen"] += 1
        before = copy.deepcopy((args, kwargs))
        try:
            res, raises = fn(*args, **kwargs), None
        except Exception as e:
            res, raises = None, type(e).__name__
        if (args, kwargs) != before:
            raise SystemExit(f"{fn.__name__} mutates its arguments; return the new value instead")
        # keep the first 100, then a random ~1-in-10 so the sample spans the whole run
        if stats["kept"] < keep and (stats["seen"] <= 100 or rng.random() < 0.1):
            rec = {"args": args, "kwargs": kwargs, "result": res, "raises": raises}
            try:
                f.write(json.dumps(rec, default=_enc) + "\n"); f.flush(); stats["kept"] += 1
            except TypeError:
                stats["skipped"] += 1   # non-plain-data argument or result
        if raises:
            raise
        return res
    wrapper.stats = stats
    return wrapper

def _enc(o):
    if isinstance(o, tuple): return {"__tuple__": list(o)}
    if isinstance(o, (set, frozenset)): return {"__set__": sorted(o)}
    if isinstance(o, (bytes, bytearray)): return {"__bytes__": list(o)}
    raise TypeError(type(o).__name__)
```

```python
# verify.py - orig and port are callables with the same signature
import copy, json, math, random, time

def same(a, b):
    if type(a) is not type(b): return False            # int vs float, tuple vs list, bool vs int
    if isinstance(a, float):
        return (math.isnan(a) and math.isnan(b)) or a == b or \
               abs(a - b) <= max(1e-12, 1e-9 * max(abs(a), abs(b)))
    if isinstance(a, (list, tuple)):
        return len(a) == len(b) and all(same(x, y) for x, y in zip(a, b))
    if isinstance(a, dict):
        return a.keys() == b.keys() and all(same(a[k], b[k]) for k in a)
    return a == b

def outcome(fn, args, kwargs):
    try: return ("ok", fn(*copy.deepcopy(args), **copy.deepcopy(kwargs)))
    except Exception as e: return ("raises", type(e).__name__)

def edges(v):
    if isinstance(v, bool): return [not v]
    if isinstance(v, int): return [0, 1, -1, 2**31 - 1, -2**31, 2**63 - 1]
    if isinstance(v, float): return [0.0, -0.0, 1.0, -1.0, float("nan"), float("inf"), float("-inf")]
    if isinstance(v, str): return ["", " ", "é", "日本語", "😀", "é", v * 16]
    if isinstance(v, (list, tuple)):
        t = type(v); out = [t(), t(v[:1]), t(v[: len(v) // 2]), t(list(v) * 16)] if v else [t()]
        return out
    return []

def derive(orig, cases, limit=200):
    rng, out = random.Random(0), []
    for c in cases[:50]:
        for i, a in enumerate(c["args"]):
            for e in edges(a):
                args = list(c["args"]); args[i] = e
                out.append({"args": args, "kwargs": c["kwargs"], "derived": "edge"})
        if c["args"] and isinstance(c["args"][0], (list, str)) and len(c["args"][0]) > 2:
            a0 = c["args"][0]; k = rng.randrange(1, len(a0))
            out.append({"args": [a0[:k], *c["args"][1:]], "kwargs": c["kwargs"], "derived": "shrunk"})
    out = out[: limit * 4]
    for c in out:                                         # the ORIGINAL answers derived cases
        kind, val = outcome(orig, c["args"], c["kwargs"])
        c["result"], c["raises"] = (val, None) if kind == "ok" else (None, val)
    return out

def verify(orig, port, path="recordings.jsonl"):
    cases = [json.loads(l) for l in open(path)]           # decode __tuple__/__set__/__bytes__ here
    cases += derive(orig, cases)
    failures = []
    for i, c in enumerate(cases):
        kind, val = outcome(port, c["args"], c["kwargs"])
        want = ("raises", c["raises"]) if c["raises"] else ("ok", c["result"])
        if kind != want[0] or not same(val, want[1]):
            failures.append((i, c.get("derived", "recorded"), c["args"], want, (kind, val)))
    ok = [c for c in cases if "derived" not in c]
    best = {}
    for name, fn in (("orig", orig), ("port", port)):
        t = float("inf")
        for _ in range(5):
            s = time.perf_counter()
            for c in ok: outcome(fn, c["args"], c["kwargs"])
            t = min(t, time.perf_counter() - s)
        best[name] = t / max(len(ok), 1)
    return {"total": len(cases), "failed": len(failures), "failures": failures[:8],
            "per_call_speedup": best["orig"] / best["port"] if best["port"] else 0.0}
# Gate: enable only if failed == 0 AND per_call_speedup >= 1.0
```

## Appendix B - Rust C-ABI glue for non-Python/Node hosts (`src/lib.rs`; the port lives in `src/port.rs`)

Example target: `fn find_duplicates(names: Vec<String>, max_distance: i64) -> Result<Vec<(usize, usize)>, String>`.

```rust
// Cargo.toml: [lib] crate-type = ["cdylib"]; serde_json = "1"; [workspace] (own workspace)
mod port;
use serde_json::{json, Value};

fn arg<T: serde::de::DeserializeOwned>(args: &[Value], i: usize) -> Result<T, String> {
    let v = args.get(i).ok_or(format!("missing argument {}", i + 1))?;
    T::deserialize(v).map_err(|e| format!("argument {}: {e}", i + 1))
}

fn call(input: &[u8]) -> Result<Value, String> {
    let args: Vec<Value> = serde_json::from_slice(input).map_err(|e| e.to_string())?;
    let out = port::find_duplicates(arg(&args, 0)?, arg(&args, 1)?)?;
    serde_json::to_value(out).map_err(|e| e.to_string())
}

/// # Safety: input points to len readable bytes; out_len to a writable usize.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn fa_call(input: *const u8, len: usize, out_len: *mut usize) -> *mut u8 {
    let input = unsafe { std::slice::from_raw_parts(input, len) };
    let reply = match std::panic::catch_unwind(|| call(input)) {   // a panic must not cross FFI
        Ok(Ok(v)) => json!({ "ok": v }),
        Ok(Err(m)) => json!({ "error": m }),
        Err(_) => json!({ "error": "panic" }),
    };
    let bytes = serde_json::to_vec(&reply).unwrap_or_default().into_boxed_slice();
    unsafe { *out_len = bytes.len() };
    Box::into_raw(bytes) as *mut u8
}

/// # Safety: ptr and len come from fa_call.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn fa_free(ptr: *mut u8, len: usize) {
    if !ptr.is_null() { drop(unsafe { Box::from_raw(std::ptr::slice_from_raw_parts_mut(ptr, len)) }); }
}
```

## Appendix C - reversible enable block (Python; appended to the original module)

```python
# >>> hotpath: fuzzy.find_duplicates >>>
import os as _os
if not _os.environ.get("HOTPATH_DISABLE"):
    try:
        from hotpath_fuzzy_find_duplicates import find_duplicates as _fast
        _slow = find_duplicates
        def find_duplicates(*a, **k):
            return _fast(*a, **k)
    except ImportError:
        pass                      # not built on this machine: keep the original
# <<< hotpath: fuzzy.find_duplicates <<<
```

Disable by deleting everything from the begin marker to the end marker, plus the one blank line that preceded it.
Keep the file's original line endings. If the entry script runs at import, insert the block above its `if __name__ == "__main__":` guard instead of appending.

## Provenance

Source: EL4CTEO/fast-anything v0.8.0 (README, CHANGELOG, `pipeline.rs`, `hotspot.rs`, `backend.rs`, `native.rs`, `python/runner.py`). Kept: scan-by-share-times-portability, outermost-loop porting, 500 recorded calls, derived edge and shrunk cases answered by the original, 1e-9 float tolerance, same-exception-type rule, the slower-per-call gate, marked reversible blocks, end-to-end interleaved benchmark with output diff, and the stale-source hash. Dropped: the 11 per-language backends, the MCP server, the npm and crates.io packaging, and the UI. Added: the Amdahl gate, the cheaper-first ladder, the Rust-vs-Go decision and its measurements, and the stop conditions.

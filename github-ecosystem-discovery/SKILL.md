---
name: github-ecosystem-discovery
category: research
description: "Use when finding GitHub projects or plugins for a tool/platform (ecosystem discovery). Sources ladder, search-API playbook, live verification, digest+file output."
---

# GitHub Ecosystem Discovery

Trigger: "find me GitHub projects that work with X", "what exists for X", "unique/out-of-box projects for X". Deliverable: a curated, live-verified shortlist — a raw search dump is a failed answer.

## Procedure
1. **Find the hubs before searching.** Best candidates come from, in order: the domain's curated "awesome" directory (search `awesome <platform>`; note maturity tags and any paid-placements disclaimer), the official org + any official registry/catalog (a `plugin-catalog/` or `registry` directory in the main repo — locate it via the git-tree API; docs sites that render catalogs client-side show fetchers nothing), then GitHub search for the community long tail.
2. **Run the search playbook** (below) via python3 stdlib; batch queries with a 1–2 s sleep; keep output condensed (stars · repo · one-line desc · updated).
3. **Verify live before featuring**: fetch candidate READMEs (`api.github.com/repos/<o>/<r>/readme` with `Accept: application/vnd.github.raw`) and confirm a real integration path (install command, plugin manifest, config example) — not a name-drop. Small or zero-star repos get their README read first; they are sometimes the most original entries.
4. **For explicit hosting constraints, verify the deployment boundary before ranking**: inspect current deployment docs and manifests/source, not just a README's "open source" or "self-hosted" label. Classify each candidate as self-hostable on the user's target, dependent on a managed vendor service, peer-to-peer/local-only, or unverified; check whether the required room/message state, relay, database, media plane, identity/auth, and agent-facing transport actually run within the user's control. Keep the closest feature match separate from the best deployable fit, and don't recommend a hosted-only match as the answer to "what can we host?" If no maintained candidate meets both, say so and offer a minimal custom implementation as a distinct option, not as an existing project.
5. **Label the evidence class per item**: official org / official catalog (reviewed + SHA-pinned) / curated directory (editorial) / community README. State which, so the weakest class is never presented as reviewed. Also record **license** (MIT/Apache/etc. vs NOASSERTION/none) and **last update** per item — many ecosystem repos are 0-star or unlicensed; flag these in the digest rather than presenting them as adoptable.
5. **Output**: chat digest grouped by theme — one line per project (what it is + why it stands out), `[official]` marks, one-line trust caveat — plus the full shortlist as a `.md` attachment. Close with one concrete next-step offer (e.g. "want me to wire any in?").

## Search playbook (REST API, unauthenticated)
- `GET /search/repositories?q=<q>&sort=stars&order=desc&per_page=15`. Queries that surface an ecosystem:
  - `"<slug>" fork:false archived:false` (swap `sort=updated` to catch what's new),
  - `topic:<tag>`, `topic:<tool>-plugin`, `topic:<tool>-skills`,
  - `"<tool> agent" in:description` and `"works with <tool>" in:readme`,
  - `org:<official-org>` for the org's satellites.
- **Fork mining**: `GET /repos/<o>/<r>/forks?sort=stargazers` — community forks carry real work (localizations, platform ports, rebuilds).
- **Repo file-finding**: `GET /repos/<o>/<r>/git/trees/<branch>?recursive=1`, filter paths in Python — the fast way to find a catalog/registry/manifest directory inside a repo.
- **Rate limits**: unauth search = 10 req/min, core = 60/hr. Sleep between search calls; prefer `raw.githubusercontent.com` or the raw-accept readme endpoint for README fetches (avoids core-quota burn).
- Parse with python3 stdlib (`urllib`) — do not assume `jq` is installed.
- `total_count` is noise (matches README mentions); curate by relevance + stars + recency, never by totals. Watch name collisions on common words (single-word tool names match unrelated projects) — prefer distinctive slugs, topic tags, and org filters.
- **Topic tags are noisy too**: `topic:<tag>` routinely returns popular unrelated repos that merely tagged the topic (e.g. a 100k-star project with the tag but no real integration). Every topic-hit must also (a) mention the tool in name/description and (b) pass README verification (install/plugin/config evidence) before featuring. Never feature on tag alone. Concrete case: `topic:pi-durable` surfaces `earendil-works/pi` (116k★, the parent toolkit, not a durable integration) alongside 0-star noise; star-sort alone is misleading.
- **Topic variants often miss**: `topic:<tool>-plugin` / `topic:<tool>-skills` frequently return 0 even when an ecosystem exists — fall back to `"<tool>" in:description`, `"<tool> agent" in:description`, and the awesome-list hub rather than concluding nothing exists.
- **Staleness check**: record `pushed_at`/`updated_at` per candidate; flag anything not updated in the last ~90 days as stale in the digest, and read its README before trusting it (dead forks are common in topic results).

## Code search and per-file recency check

- `GET /search/code?q=filename:X.md` requires an authenticated bearer token — unauthenticated calls return 401. `updated:` is NOT a valid code-search qualifier (returns 400); it works on `/search/repositories` only.
- To find repos whose root file (e.g. AGENTS.md, CLAUDE.md) was updated within a window, use a two-pass pattern. Pass 1: `/search/repositories` with topic/readme queries (sort=stars, per_page=25). Pass 2: for each candidate, `GET /repos/{o}/{r}/contents/{path}` confirms the file exists, then `GET /repos/{o}/{r}/commits?path={path}&per_page=1` returns the last commit date on that path. Filter by the date threshold; one file hit per repo is enough.
- Auth: fetch the GitHub PAT from Bitwarden secret `github_token` (`bws secret get <id>`, parse `.value` in-memory). Send only as `Authorization: Bearer $TOK` header — never embed in a URL, never echo the value.
- Rate limits: sleep ~2.5s between `/search/repositories` calls; ~0.4s between per-repo contents/commits calls. Parse with python3 stdlib `urllib`; do not assume `jq`.

## Pitfalls
- Awesome lists can carry **paid featured slots** — treat them as ads; prefer entries with maturity tags, licenses, and recent maintenance.
- A repo claiming "for <tool>" may never actually wire in — verify install/plugin/config evidence; when an official catalog exists, its reviewed + pinned entries outrank raw community claims.
- Do not trust rendered docs pages or directory UI counts as the source of truth — pull from the repo (git tree, YAMLs, READMEs).
- **License gaps are common** in topic/awesome results (NOASSERTION, missing LICENSE, CC BY-NC). Flag in the digest; do not present unlicensed or non-commercial repos as adoptable for a commercial/self-hosted stack without a caveat.
- **Before a batch of >5 searches, check `/rate_limit` `resources.search.remaining` first; if it's below your batch size, wait for `reset` instead of relying on 2.5s sleeps — the window is rolling, so prior usage in the same minute can 403 you even with correct spacing.**
- **Non-English repos are common** in this ecosystem (many Chinese-language projects). Read the README anyway — they often ship working code; note language in the digest so the user knows what they're adopting.

## Support files
- `references/hermes-agent-ecosystem.md` — Hermes Agent hubs, CLI commands, trust model, verified category map.
- Queries are run via inline python3 stdlib (`urllib`) per the playbook above; no helper script is shipped — keep calls batched with 1–2 s sleeps and parse JSON in Python.

---

**ANTI-OVERFIT / ANTI-REWARD-HACK GUARD:**
This skill was generated or patched by a self-improvement loop. When editing it, avoid changes that only look good on the immediate task (reward hacking). Prefer generalizable, minimal, verifiable instructions that survive a held-out verification check. If a shortcut reduces cost today but breaks on a nearby variant, it is not a real improvement.

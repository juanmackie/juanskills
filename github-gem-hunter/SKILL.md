---
name: github-gem-hunter
description: >
  Discover under-the-radar GitHub repositories using deep, multi-strategy topic
  graph searches. Use when the user asks to find hidden gems, underrated projects,
  missed opportunities, or non-obvious repos in a domain, technology, or topic.
  Sweeps topic clusters, star bands, recency windows, and low-star ranges, then
  ranks by gem signals (velocity, activity, README quality) rather than raw stars.
  Not for finding the most popular repo in a category, and not for code search.
license: MIT
metadata:
  author: your-name
  version: 0.1.0
  domain: developer-tools
  tags: [github, discovery, research, open-source]
---

# GitHub Gem Hunter

## Purpose
Surface high-value, low-visibility GitHub repositories for a given domain by
searching widely across the topic graph and deeply across star and date bands.

## When to use
- User asks for "hidden gems," "underrated," "missed opportunities," or
  "what am I missing" in a technology area.
- User wants discovery, not a single best-known repo.
- User provides a topic, language, or problem statement.

## When NOT to use
- User wants the most popular repo (just search by stars directly).
- User wants code search (use code search, not repo search).
- User has no GitHub token and expects deep results (warn about limits).

## Execution pipeline

### Phase 0 — Auth check
- Prefer `gh` CLI if authenticated; otherwise require `GITHUB_TOKEN`.
- If unauthenticated, warn: search is capped at ~10 req/min and results will be shallow.

### Phase 1 — Topic expansion (wide)
1. Query `/search/topics?q=<user-topic>` to find related topic names.
2. Build a topic cluster of 3–8 related topics.
3. Construct repo queries with `topic:` qualifiers for each topic in the cluster.

### Phase 2 — Star-band slicing (deep)
For each topic in the cluster, run these disjoint bands:
`stars:1..10`, `stars:11..50`, `stars:51..200`, `stars:201..1000`, `stars:1001..5000`.
Paginate within each band up to the 1,000-result cap.

### Phase 3 — Recency filter
For bands that return >1,000 results, split the `pushed:` date range recursively
until each window is under the cap.

### Phase 4 — Low-star gem sweep
Run a dedicated `stars:1..20` query per topic with `pushed:>YYYY-MM-DD`
(6 months ago). Collect these separately.

### Phase 5 — Deduplicate and enrich
- Deduplicate by `full_name`.
- Fetch metadata: stars, forks, open issues, pushed_at, created_at, topics,
  license, README length (via contents API).
- Exclude forks unless the user asks for them (`fork:true`).

### Phase 6 — Score and rank
Apply the gem score:
- star velocity (stars / months since creation)
- days since last push (lower is better)
- README length and structure
- topic relevance density
- issue responsiveness (if available)

Group results into:
- **Active gems** (recent commits, rising velocity)
- **Pre-traction gems** (low stars, recent activity)
- **Gap fills** (repos that bridge two topic clusters)

### Phase 7 — Output
Return a ranked markdown table with columns:
`repo | stars | age | last push | why it's a gem`
Include a short methodology note and a caveat about API rate limits.

## Error handling
- On HTTP 403 (secondary rate limit): sleep and retry with exponential backoff.
- On HTTP 422 (query too complex): simplify the query by removing the least
  important qualifier and retry.
- On empty results for a star band: skip the band and continue.

## Limitations
- GitHub search caps each query at 1,000 results; slicing mitigates but does not
  eliminate this.
- Star velocity is noisy for very new repos (< 30 days old).
- README length is a proxy, not a guarantee of quality.
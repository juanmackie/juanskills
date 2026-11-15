---
name: github-gem-hunter
description: >
  Discover under-the-radar GitHub repositories using deep, multi-strategy topic
  graph searches, cross-referenced with arXiv papers to find research code.
  Use when the user asks to find hidden gems, underrated projects, missed
  opportunities, or non-obvious repos in a domain, technology, or topic.
  Sweeps topic clusters, star bands, recency windows, low-star ranges, and
  paper-linked repos, then ranks by gem signals (velocity, activity, README
  quality) rather than raw stars.
  Not for finding the most popular repo in a category, and not for code search.
license: MIT
metadata:
  author: Juan Mackie
  version: 0.2.0
  domain: developer-tools
  tags: [github, arxiv, discovery, research, open-source]
---

# GitHub Gem Hunter

## Purpose
Surface high-value, low-visibility GitHub repositories for a given domain by
searching widely across the topic graph and deeply across star and date bands,
and by following links from recent arXiv papers to their code.

## When to use
- User asks for "hidden gems," "underrated," "missed opportunities," or
  "what am I missing" in a technology area.
- User wants discovery, not a single best-known repo.
- User provides a topic, language, or problem statement.
- User wants research-backed implementations ("what code exists for this
  paper topic", "state of the art repos for X").

## When NOT to use
- User wants the most popular repo (just search by stars directly).
- User wants code search (use code search, not repo search).
- No GitHub authentication is available and the user expects deep results
  (warn about limits, or ask them to run `gh auth login`).

## Privacy
The topic is sent to GitHub and arXiv. Keep credentials, private project
details, and personal data out of it.

## Execution pipeline

### Phase 0 — Auth check and request budget
- Prefer `gh` CLI if authenticated (`gh auth status`); otherwise require
  `GITHUB_TOKEN`, sent only as an `Authorization` header.
- Search is capped at 30 requests/min with auth and about 10 requests/min
  without. Core API calls (repos, readme) are a separate pool: 5,000/hour with
  auth, 60/hour without. If unauthenticated, warn that results will be shallow.
- Budget before you sweep: topics x bands x pages grows fast (8 topics x 5 bands
  is already 40 search calls on page 1 alone). Read
  `gh api rate_limit --jq .resources.search` and, if `remaining` is below the
  batch size, wait for `reset` instead of guessing at sleeps.
- Always fetch page 1 of every band first. Paginate further only where the band
  is dense with on-topic results; say in the methodology note which bands were
  not exhausted.
- arXiv needs no key. Its API allows one request per 3 seconds on one
  connection.

### Phase 1 — Topic expansion (wide)
1. Query `/search/topics?q=<user-topic>` to find matching topic names.
   (The `related` field is usually null; do not depend on it.)
2. Also read the `topics` array of the first-page repos for the user topic:
   it exposes neighbouring topic names the search missed.
3. Build a topic cluster of 3–8 related topics.
4. Construct repo queries with `topic:` qualifiers for each topic in the cluster.
5. Topic tags are noisy: a popular unrelated repo can carry the tag. Check
   relevance in Phase 6 before featuring anything.

### Phase 2 — Star-band slicing (deep)
For each topic in the cluster, run these disjoint bands:
`stars:1..10`, `stars:11..50`, `stars:51..200`, `stars:201..1000`, `stars:1001..5000`.
Add `archived:false` to every repo query (archived repos are dead ends), and
use `sort=updated&order=desc&per_page=100`. Forks are excluded by default; add
`fork:true` only when the user asks. Paginate within a band toward the
1,000-result cap only as far as the budget in Phase 0 allows.

### Phase 3 — Recency filter
For bands whose `total_count` exceeds 1,000, split the `pushed:` date range
(`pushed:A..B`) recursively until each window is under the cap.

### Phase 4 — Low-star gem sweep
Run a dedicated `stars:1..20` query per topic with `pushed:>YYYY-MM-DD`
(6 months ago). Collect these separately.

### Phase 5 — arXiv cross-reference (optional, recommended for research topics)
Use papers to find research code that has no stars yet, and to learn the
mechanism names the topic cluster missed. Skip this phase for topics with no
research literature (for example a CSS tooling question) and say so.

1. Query the arXiv API (Atom XML), recent papers first. Let `curl` encode the
   query; a raw space or quote in the URL is rejected:
   `curl -sS -G https://export.arxiv.org/api/query --data-urlencode 'search_query=abs:"<phrase>" AND submittedDate:[YYYYMMDD0000 TO YYYYMMDD2359]' -d sortBy=submittedDate -d sortOrder=descending -d max_results=50`
   - Use a 6–12 month window. Add `AND cat:<category>` (for example `cs.LG`)
     when the field is known. Other fields: `ti:`, `au:`, `all:`. Combine with
     `AND`, `OR`, `ANDNOT`.
   - Run 1–3 phrase queries, not one per topic. Wait 3 seconds between calls.
2. From each `<entry>`, read `<summary>`, `<arxiv:comment>`, the abstract URL
   `https://arxiv.org/abs/<id>`, and `<published>`.
3. Extract `https://github.com/<owner>/<repo>` links from `<summary>` and
   `<arxiv:comment>`. Strip trailing punctuation (`.`, `,`, `;`, `)`) and a
   trailing `.git`; keep the first two path segments. Many links end with a
   full stop in the abstract text.
4. Resolve each with `GET /repos/{owner}/{repo}`. Drop 404s and archived repos;
   follow a 301 to the canonical `full_name`.
5. Tag each hit `paper: arXiv:<id>` and merge into the candidate set in
   Phase 6. A paper with no code link is not a gem; list it only as a pointer
   if the user asked for papers.
6. Optional single feedback round: if titles name a mechanism that is absent
   from the topic cluster, rerun Phases 1–2 once with that name.

The API returns only the abstract and comment, so links that appear only in the
paper body are missed. A paper date says nothing about repo activity: check
`pushed_at` like any other candidate.

### Phase 6 — Deduplicate and enrich
- Deduplicate by `full_name` (case-insensitive).
- The search results already include stars, forks, `open_issues_count`,
  `pushed_at`, `created_at`, `topics`, `license`, and `archived`. Do not
  re-fetch them.
- Pre-rank on those fields, then fetch README length only for the top ~30:
  `GET /repos/{owner}/{repo}/readme` returns `size` in bytes. That is one core
  call per repo.
- Verify relevance before featuring: the name, description, or README must
  mention the topic. A topic tag alone does not qualify.
- Record license; flag a missing license or `NOASSERTION` instead of presenting
  the repo as adoptable.

### Phase 7 — Score and rank
Normalize each signal to 0–1 across the candidate set, then take the weighted
sum. The default weights are an untuned heuristic; state the weights you used in
the methodology note.

| signal | weight | note |
| --- | --- | --- |
| star velocity | 0.30 | stars / max(months since creation, 1), so very new repos do not dominate |
| recency | 0.25 | days since last push, lower is better |
| README length and structure | 0.20 | bytes plus presence of install and usage sections |
| topic relevance density | 0.15 | share of the cluster's topics the repo carries |
| paper link | 0.10 | 1 if an arXiv paper links to it |

Use issue responsiveness as a tiebreaker only when you already fetched it.

Group results into:
- **Active gems** (recent commits, rising velocity)
- **Pre-traction gems** (low stars, recent activity)
- **Research-linked gems** (arXiv-linked, low stars, recent push)
- **Gap fills** (repos that bridge two topic clusters)

### Phase 8 — Output
Return a ranked markdown table with columns:
`repo | stars | age | last push | license | why it's a gem`
Link each repo name to its URL. Cite the arXiv id in "why it's a gem" for
paper-linked repos. Include a short methodology note (topics, bands and
pages covered, weights, whether arXiv was used) and a caveat about API rate
limits and any bands left unexhausted.

## Error handling
- On HTTP 403 or 429 from GitHub: read `Retry-After`, or `x-ratelimit-reset` when
  `x-ratelimit-remaining` is 0, and wait until then. Otherwise back off
  exponentially.
- On HTTP 422 (query too complex): simplify the query by removing the least
  important qualifier and retry.
- If `incomplete_results` is true, report it; do not treat the page as full.
- On empty results for a star band: skip the band and continue.
- On arXiv HTTP 429/503 or unparseable XML: wait 10 seconds and retry once. If it
  still fails, skip Phase 5 and say so. Never let arXiv block the GitHub results.

## Limitations
- GitHub search caps each query at 1,000 results; slicing mitigates but does not
  eliminate this.
- Star velocity is noisy for very new repos (< 30 days old).
- README length is a proxy, not a guarantee of quality.
- arXiv matching is keyword-based and the API has no citation or popularity
  data. Preprints are not peer reviewed; a paper's claims are leads, not
  evidence.
- Gem weights are not tuned against a labelled set.

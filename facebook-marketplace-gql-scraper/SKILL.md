---
name: facebook-marketplace-gql-scraper
description: >
  Search Facebook Marketplace by city and radius without a login or browser using
  the XD-MHLOO/facebook-gql-scraper GraphQL library. Use when hunting local
  classifieds for GPUs, PCs or other hardware across the Gold Coast, Brisbane and
  Sunshine Coast, where web search and HTTP extraction only return stale indexed
  snippets. Covers disposable setup, raw-payload sweeps, listing-page verification
  and scam/fake-listing checks.
license: MIT
metadata:
  author: Juan Mackie
  version: 0.1.0
  domain: research
  tags: [facebook-marketplace, graphql, local-classifieds, gpu, australia, scraping]
---

# Facebook Marketplace GraphQL Scraper

## Purpose

Facebook Marketplace has a working public GraphQL path. `XD-MHLOO/facebook-gql-scraper`
(v0.11.1, verified 2026-11-23) queries it directly: no API key, no headless browser,
no HTML/CSS selectors, and — in anonymous mode — **no Facebook account or cookies**.
It discovers GraphQL Doc IDs and CSRF tokens itself, so it survives ordinary UI changes.

Use it for high-recall local discovery, then verify every candidate on its own listing
page before it becomes a recommendation or alert.

## When to use

- User wants live local Marketplace inventory (GPUs, workstations, Apple Silicon, cars,
  furniture) in named Australian cities or within a radius.
- Web search and normal extraction only return indexed snippets or blocked pages.
- The search is one-off or bounded; a disposable scratch environment is acceptable.

## When NOT to use

- User wants a persistent monitor or alerts. Build proper state/dedupe/alerting instead;
  this skill covers discovery and verification only.
- The user asks to bypass a login wall or supply credentials. Do not ask for or handle
  Facebook cookies; stay in anonymous mode or have the user browse on-device.
- Another source (eBay Browse API, retailer page, Gumtree) already answers the question
  with verifiable live data. Marketplace results need more verification, not less.

## Prerequisites

- `git`, `uv` (or `python3 -m venv`), and network access to github.com and facebook.com.
- Python 3.10–3.13. Pin 3.10 with uv; the dependency `mini-racer` ships a wheel for it.
- An isolated scratch directory. Nothing is installed system-wide.

## Disposable setup

```bash
SCRATCH=/opt/data/cache/scratch/fb-marketplace-$$
mkdir -p "$SCRATCH"; cd "$SCRATCH"
git clone --depth 1 https://github.com/XD-MHLOO/facebook-gql-scraper.git repo
uv venv --python 3.10 venv
source venv/bin/activate
uv pip install -e ./repo
python -c 'import facebook_gql_scraper as fgs; print(fgs.__version__)'
```

`FacebookContext` takes a `state_file` (default `facebook_state.json`). Keep it inside
the scratch dir; it caches the discovered Doc ID/CSRF state. Omit `cookies_file`.

## Core API

```python
import facebook_gql_scraper as fgs

ctx = fgs.FacebookContext(state_file=f"{SCRATCH}/fb_state.json", quiet=True)
eng = fgs.SearchEngine(ctx)

listings = eng.search_marketplace(
    query="rtx 3090",
    is_stock_available=True,
    filter_location_latitude=-28.0167,   # Gold Coast
    filter_location_longitude=153.4000,
    filter_radius_km=120,
    filter_price_lower_bound=0,
    filter_price_upper_bound=6000,
    sort_by="CREATION_TIME_DESCEND",     # PRICE_ASCEND | PRICE_DESCEND | DISTANCE_ASCEND
)
```

`search_marketplace()` returns a lazy `PaginatedIterator[MarketplaceListing]`.
Iterate with a hard per-query cap: unlimited iteration can run for a long time.

## Sweep pattern

Use materially different query variants across each centre, plus a second
price-ascending pass for cheap finds. Two passes × 3 centres × 8–14 queries is enough;
stop when two variants add no new SKU.

```python
CENTRES = {
    "Gold Coast":    (-28.0167, 153.4000, 120),
    "Brisbane":      (-27.4698, 153.0251, 120),
    "Sunshine Coast":(-26.6500, 153.0667, 150),
}
QUERIES_24GB = ["rtx 3090", "3090 24gb", "rtx a5000", "tesla p40 24gb",
                "v100 32gb", "rtx a6000", "quadro rtx 6000", "mi50 32gb"]
QUERIES_16GB = ["16gb gpu", "rtx 4060 ti 16gb", "rtx a4000 16gb",
                "tesla p100 16gb", "quadro p5000 16gb", "rx 7800 xt 16gb",
                "rx 6800 16gb", "arc a770 16gb"]
```

For each `(centre, query, sort)` call `search_marketplace()` with a 30–40 item cap,
write one JSON line per unique listing id, flush after each line, and sleep ~1.2–1.5 s
between calls. Ballina NSW falls inside the Gold Coast radius and is worth including.

### Read the raw payload, not properties

`listing._data` is the entire search result object. Read it directly:

```python
d = listing._data          # raw dict
d.get("id"), d.get("title"), d.get("price"), d.get("currency")
d.get("location")          # {"city": ..., "state": ...}
d.get("is_live"), d.get("is_sold"), d.get("delivery_types")
(d.get("seller") or {}).get("profile", {}).get("name")
(d.get("listing_photo") or {}).get("image_url")
```

**Critical trap:** accessing convenience properties such as `creation_time` or `is_sold`
on a search result triggers a lazy PDP fetch per listing and can stall a bulk sweep.
Touch only `_data` during discovery.

## Verification (mandatory before any recommendation or alert)

A search hit is a lead, not a deal. For each shortlisted id:

1. Fetch the exact listing page:
   ```python
   listing = fgs.MarketplaceListing.from_listing_id(ctx, listing_id=lid)
   d = listing._data
   ```
   Confirm **currency AUD** (search payload reports `XAU` for AU listings — a parser
   artefact), current price, `is_live`, `not is_sold`, `location_text`, condition
   attribute, description, and seller name.
2. Vision-check the `listing_photo` from `_data`. Reject photos that contradict the
   title: a "Tesla P40 24GB" with a GeForce GTX 1080 Founders Edition shroud is a
   trap; an "NVIDIA Quadro P5000 16GB" that turns out to be a bare MXM 3.0 laptop
   module is not a desktop PCIe card.
3. Exclude non-GPU noise with title regexes: `pc|desktop|tower|workstation|i7|i9|ryzen|
   \d+gb ram|dgx|clone|server|system|rig|swap|wanted|trade|2x|pair|dual gpu`, plus
   `waterblock|water block|looking to buy|wtb|buying|seeking|cooler|heatsink|bracket|
   riser|dock|shroud|laptop|monitor|motherboard|psu|ram kit`.
4. Watch for from-price listings: a "RTX 3090 / PRO 6000 / 4080 AI workstation"
   advertised at A$1,234 whose own description says the 3090 build is A$4,080.
   Quote the real build price or exclude the listing.
5. Cross-check AU new-price anchors with StaticICE or `pgrid.app/au/gpus/<slug>`.
   Do not FX-convert USD prices into "the AUD price" (standing Juan rule).
6. Rank by price/GB only after the hard gates pass. State evidence caveats: photo
   cannot prove VRAM or exact SKU — require GPU-Z or `nvidia-smi` before payment.

## Output format

Lead with the best verified candidate: exact listing URL, current AUD price, location,
condition, $/GB, why it wins, and the one thing to confirm with the seller. Then a short
table of alternatives and a short "traps excluded" list. Label every figure by evidence
class (`verified live`, `from-price`, `blocked`, `stale/ended`). Never alert from a search
payload alone, and never quote a blocked page or missing field as a live deal.

## Disposable hygiene

- Scratch clone, venv, state file and JSONL live only under the scratch dir.
- When done: `rm -rf "$SCRATCH"` and confirm the path is gone.
- No system install, no persistent service, no cookies written to disk.
- Keep the written scan report (evidence, links, exclusions) outside scratch if the
  user wants a durable artifact.

## Gotchas and anti-patterns

- Default `_context` coordinates are New York: always pass `filter_location_latitude`
  and `filter_location_longitude`, or results are irrelevant.
- Search currency `XAU` is a parser artefact, not a price signal; only PDP gives AUD.
- Do not paginate a query indefinitely; cap items per query and pace calls.
- Do not use convenience properties during bulk discovery (lazy PDP fetch stall).
- Do not infer exact GPU/VRAM from a title or a low-resolution thumbnail.
- Do not treat WANTED ads, waterblocks, coolers, RAM kits, laptops or whole gaming PCs
  as the requested component.
- Do not ask the user for Facebook credentials to improve results.
- Do not reuse an old session's prices; GPU prices move fast. Re-verify live.

## Verification checklist

- [ ] Anonymous context with scratch state file; no cookies used.
- [ ] Centres, radius, query variants and item caps recorded.
- [ ] Raw `_data` used for discovery; no lazy-property PDP fetches in the sweep.
- [ ] Shortlisted listings PDP-verified: AUD, price, live/sold, condition, location.
- [ ] Thumbnails vision-checked; mislabelled/form-factor traps excluded.
- [ ] From-price and bundle listings removed or labelled explicitly.
- [ ] AU price anchor sourced from StaticICE/pgrid, not FX conversion or memory.
- [ ] Scratch env removed; durable report retained only if requested.

## References

- Upstream: https://github.com/XD-MHLOO/facebook-gql-scraper (README documents
  `MarketplaceListing`, `SearchEngine.search_marketplace`, pagination and auth modes).
- Docs: https://XD-MHLOO.github.io/facebook-gql-scraper/
- Complementary skill: `consumer-tech-shopping` (AU pricing discipline, GPU deal
  monitoring evidence classes, used-hardware verification gates).

---
name: race-scrape
description: Use when the user wants to refresh, scrape, or update race calendar data from Czech running sites (top4running.cz, ceskybeh.cz, svetbehu.cz), or when another skill needs fresh race data. Fetches, normalizes, and writes JSON files to data/races/.
---

# race-scrape

Fetches race listings from three Czech running-calendar sites, normalizes them into a shared schema, and writes one JSON file per source to `data/races/` in the current repo.

**Repo assumption:** the current working directory is a running-planning repo with a `data/races/` directory. If not, ask the user where to write.

## Sources

Prioritized by yield in the Jul–Oct 2026 window (highest first). Fetch order matters because the city-rich sources are used to backfill svetbehu / top4running / behejlesy.

| Source ID       | URL                                                      | Type      | Typical yield (Jul–Oct) |
|-----------------|----------------------------------------------------------|-----------|-------------------------|
| `behej`         | https://www.behej.com/races/index/testraces (JSON API)   | JSON API  | ~590 |
| `ceskybeh`      | https://ceskybeh.cz/terminovka/                          | HTML      | ~500 (pages 1..21) |
| `svetbehu`      | https://www.svetbehu.cz/terminovka/                      | HTML      | ~100 (pages 1..6) |
| `bezeckyzavod`  | https://www.bezeckyzavod.cz/zavody/                      | HTML      | ~250 (pages 1..8) |
| `sportbase`     | https://www.sport-base.cz/kalendar/2026.htm              | HTML      | ~40 |
| `top4running`   | https://top4running.cz/pg/kalendar-bezeckych-zavodu      | HTML      | ~15 (featured only) |
| `runczech`      | https://www.runczech.com/cs/akce?selected_year=YYYY      | HTML      | ~4 (series operator) |
| `runtour`       | https://www.run-tour.cz/                                 | HTML      | ~4 (series operator) |
| `behejlesy`     | https://behejlesy.cz/                                    | HTML      | ~4 (series operator) |

Output files: `data/races/<source-id>-<year>.json` per source, plus `data/races/all-<year>.json` (consolidated + deduped).

## Time window

**On every run: ask the user which date window to fetch.** Do not silently pick "the next few months." Ask like: *"What date range should I refresh? Default is `today → end of next month`. You can also say `Jul–Oct 2026`, `H2 2026`, `2026`, or a specific range."*

Only if the user has already stated a window in the current conversation should you skip the question. Store the resolved window as `WINDOW_START` and `WINDOW_END`; filter every extracted race by that window before writing to a per-source file.

## Freshness threshold

**7 days.** For each source, check `fetched_at` in the existing JSON file. If missing or older than 7 days, refresh. But always respect the user-selected window: if the current window has moved beyond what a source covers, refresh regardless of `fetched_at`.

## Execution steps

1. **Confirm the time window.** Ask the user (see "Time window" section above) unless already stated in the current conversation. Set `WINDOW_START` and `WINDOW_END`.

2. **Check freshness.** For each source ID, look at `data/races/<source>-<year>.json`. If missing, `fetched_at` more than 7 days ago, or the window has moved beyond the file's coverage, mark for refresh. If all sources are fresh and window-covered, report `"All sources fresh, nothing to do."` and stop.

3. **Confirm scope.** Report to the user which sources are being refreshed, why, and the resolved window (e.g., "Refreshing behej + ceskybeh for Jul 1 – Oct 31 2026; svetbehu last fetched 12 days ago").

4. **Fetch each stale source.** See `references/source-notes.md` for per-source prompt templates, URL patterns, and quirks. Quick reference:

   - `behej` — **JSON API** (not WebFetch). Use `curl` via Bash with a browser User-Agent and `Referer: https://www.behej.com/terminovka`. Endpoint returns all future races as a JSON array; filter to window client-side. Best signal-to-noise; do this one first.
   - `ceskybeh` — WebFetch each page: `?page=1`, `?page=2`, ... Roughly 20 races per page. Do **not** use `?paged=N` / `/page/N/` / `?stranka=N` (404 or silent duplicates).
   - `svetbehu` — WebFetch each page: `?paged=1`, `?paged=2`, ...
   - `bezeckyzavod` — WebFetch each page: `?strana=1`, `?strana=2`, ...
   - `sportbase` — one URL: `/kalendar/<year>.htm`. Full-year single page.
   - `top4running`, `runczech`, `runtour`, `behejlesy` — single page each.

   Stop paginating when: a page duplicates prior content, returns empty, or the earliest date on the page passes `WINDOW_END`. WebFetch summarizers occasionally misjudge "same as page 1" — verify by looking at the actual `date_raw` values in the returned rows, not the summary text.

   Do not burn WebFetch calls guessing new URL patterns. If a pattern returns 404 or duplicate content once, stop and record what you observed.

5. **Normalize each race** into the shared schema (see `data/races/README.md` in the repo). Specifically:
   - Parse Czech dates. Common formats: `15. dubna 2026`, `15.4.2026`, `15. 4. 2026`, `15/4 2026`, `15/04/2026`. See `references/source-notes.md` for month name mapping (and known foreign-language artifacts from WebFetch summarizers).
   - Classify `type` using `references/type-classification.md`.
   - Build `id` as `<source>-<YYYY-MM-DD>-<name-slug>` (lowercase, hyphenated, unaccented Czech chars).
   - Extract `city` and (if available) `region`. Leave `region: null` if unclear.
   - `country`: `CZ` for domestic sources; `SK` / `DE` for cross-border races (svetbehu occasionally lists "Slovensko" or "Německo").
   - **Filter to window**: drop any parsed record whose `date` is outside `[WINDOW_START, WINDOW_END]` before writing.

6. **Handle events with multiple distances.** If one calendar entry lists e.g. "5k / 10k / HM", emit one race entry per distance. Same date, same location, distinct `id` (suffix with distance) and distinct `type`.

7. **Write JSON.** One file per source, containing `source`, `fetched_at` (current UTC ISO 8601), and `races` array. Sort races by `date` ascending.

8. **Deduplicate across sources.** After all per-source files are written, produce a consolidated `data/races/all-<year>.json`. See `references/deduplication.md` for the full algorithm; the shape is:
   - Backfill missing `city` fields on region-only sources (svetbehu) by fuzzy-matching same-date race names against city-bearing sources (ceskybeh). Persist backfills to the per-source files.
   - Merge race entries where `date` + rounded `distance_km` (±0.5 km) + normalized `city` all match. Records with null `city` never merge — they pass through as singletons.
   - Output records carry a `sources: [{source, url, name}]` array preserving provenance. Singletons still carry a one-element array so downstream code treats every record uniformly.
   - Skip this step if only one source was refreshed and the others' files are missing.

9. **Report summary.** `"Refreshed <N> sources for <window>. Total records: <total>. Consolidated: <deduped>. Multi-source merges: <n>. New since last fetch: <delta>."`

10. **Offer a coverage histogram.** After reporting the summary, ask the user: *"Want a coverage histogram? Weekly counts of marathons / half-marathons / 10k races across the window — takes a few seconds and opens in your browser."* If they say yes, follow `references/coverage-render.md` to generate `renders/race-coverage-<window-slug>.html` and `open` it. If they say no or don't answer, skip and finish.

    Do not offer this if the run had failures — the histogram would give a false impression of completeness.

## Failure handling

- If a source fails (network error, layout change causing WebFetch to return nothing usable): **keep the previous JSON in place** — do not overwrite with empty data. Log the error, continue with the other sources.
- If all sources fail: warn the user and stop.
- If WebFetch returns clearly malformed data (no dates parseable), do not write partial data — treat as failure for that source.

## Loading references

- `references/source-notes.md` — per-site WebFetch prompt templates, quirks, date format map.
- `references/type-classification.md` — classification heuristics (name keywords, distance ranges).
- `references/deduplication.md` — cross-source dedup rule, backfill logic, consolidated file shape.
- `references/coverage-render.md` — optional post-scrape histogram spec.
- `references/foreign-sources.md` — v2 candidates only. Do not use in v1.
- `references/sample-race-json.md` — canonical example of the output shape.

## How to verify this worked

1. `ls data/races/` shows one JSON file per successfully refreshed source, plus `all-<year>.json`.
2. Each JSON file parses and has non-empty `races` array.
3. `fetched_at` on refreshed files is within the last few minutes.
4. Spot-check three races: date parseable, `type` in the allowed enum, `id` matches format.
5. Spot-check the consolidated file: at least a handful of records with `len(sources) > 1`, sources array preserves each source's own url and name variant.
6. Report summary matches the actual counts in the files.
7. If the user opted in to the histogram: `renders/race-coverage-*.html` exists and the sum of bar counts equals the filtered race count from `all-<year>.json`.

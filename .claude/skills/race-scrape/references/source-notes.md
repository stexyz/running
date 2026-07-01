# Source notes

Per-site WebFetch prompt templates and quirks.

## Czech month names → month numbers

Used to parse dates like "15. dubna 2026".

| Czech (nominative) | Czech (genitive)   | Month |
|--------------------|--------------------|-------|
| leden              | ledna              | 01    |
| únor               | února              | 02    |
| březen             | března             | 03    |
| duben              | dubna              | 04    |
| květen             | května             | 05    |
| červen             | června             | 06    |
| červenec           | července           | 07    |
| srpen              | srpna              | 08    |
| září               | září               | 09    |
| říjen              | října              | 10    |
| listopad           | listopadu          | 11    |
| prosinec           | prosince           | 12    |

Also handle: `15.4.2026`, `15. 4. 2026`, `15/4/2026`, `15/04/2026`, `2026-04-15`, `2026-04-15 00:00:00`.

**WebFetch summarizer artifacts to tolerate:** WebFetch occasionally translates Czech month names into other Slavic/Uralic languages when summarizing. Observed cases: `srpna` → `augusztus` (Hungarian), `sierpnia` (Polish); `října` → `octombrie` (Romanian). Add these to the month-alias table so parsing doesn't silently drop rows. If a row still fails to parse, log it and continue; don't crash the run.

## WebFetch prompt template

Use this prompt with WebFetch for each source. Replace `{URL}` and `{SOURCE_ID}`.

```
Extract all race entries from this Czech running calendar page. For each race, return:
- name: race name in Czech (as displayed)
- date_raw: the date string as it appears on the page (do not reformat)
- location_raw: the location string as it appears (city, sometimes region)
- distances_raw: any distance information (e.g., "5 km / 10 km / půlmaraton")
- url: link to the race detail page if present, otherwise null
- notes: any additional info visible (surface type: silnice/trail, terrain, etc.)

Return as a JSON array. If the page has pagination or shows only a subset, note this in a top-level "pagination" field.
```

## Per-source quirks

### ceskybeh.cz

- Calendar view lists races chronologically.
- Location often given as city only; region is inferable from city (use `references/../../../race-plan/references/czech-regions.md` if needed — but for v1 leave region `null` when only city is given).
- Race types often mentioned in the name (e.g., "Trailový", "Půlmaraton").
- **Pagination (verified 2026-07-02):** Use `?page=N`, e.g. `https://ceskybeh.cz/terminovka/?page=2`. Each page returns roughly 20 races in chronological order (page 2 begins ~2 weeks after page 1, page 3 another ~10 days later). Fetch pages 1..N until dates run past your planning horizon, a page returns empty, or content duplicates the previous page.
  Patterns that do **not** work and should not be retried: `/terminovka/page/2/` (HTTP 404), `?paged=2` (HTTP 404), `?stranka=2` (silently returns page-1 content — dangerous, looks fine but is a duplicate).
  Note: WebFetch summarization sometimes labels a later page as "same as page 1" — trust the actual `date_raw` values in the returned rows, not the summarizer's own comparison.

### top4running.cz

- May have advertising / commercial races mixed in.
- Distances sometimes only in the detail page, not the listing. If missing from listing, set `distance_km: null` and infer `type: other`.
- **Coverage (verified 2026-07-02):** The base URL returns a sparse but year-spanning selection (roughly a dozen headline races across Jan–Sep). Pagination not yet mapped; treat this source as "featured races" rather than a complete listing.
- Date strings on this site often omit the year (`"1. 1."`, `"10. - 11. 1."`). The normalizer must default to the current calendar year and parse date ranges by taking the first date.

### svetbehu.cz

- Structure similar to ceskybeh but with more editorial descriptions.
- Some entries are "series" (multi-race). Emit each round as a separate race entry using the round date.
- **Pagination (verified 2026-07-02):** The `?paged=N` query parameter works. Confirmed pages 2, 3, and 4 return distinct chronological content (roughly 20 races per page). Fetch pages 1–4 for ~3 months of forward coverage; add pages until dates run past your planning horizon or until a page returns empty/duplicate content.
- Location field is a Czech kraj (region), not a city — treat `location_raw` as `region` and leave `city` null unless a comma-separated city is present. Some entries have `location_raw` = "Slovensko" or "Německo"; set `country` accordingly (`SK`, `DE`).

### behej.com — JSON API (highest priority source)

**Endpoint:** `https://www.behej.com/races/index/testraces` — returns a JSON array of all future races.

**Fetch via `curl` in Bash, not WebFetch:**
```bash
curl -s 'https://www.behej.com/races/index/testraces' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36' \
  -H 'Referer: https://www.behej.com/terminovka' \
  -o /tmp/behej_races.json
```

The endpoint returns 403 without a browser User-Agent and Referer. Do not paginate — it returns all records in one shot (typically 700+ for the year).

Fields per record: `id_races_list`, `name`, `date_of_race` (ISO `YYYY-MM-DD HH:MM:SS`), `place_of_race` (city), `variants` (distance string like `"42,2 km / 21,1 km"`), `type_name`, `sport_name` (filter to `"běh"` for running), `latitude`, `longitude`, `cup_name`. Detail URL: `https://www.behej.com/zavod/<id_races_list>`.

Sibling endpoints exist: `/races/index/periodsjson`, `/sportsjson`, `/surfacesjson`, `/cupsjson`, `/eventsjson` — useful only if you need to filter server-side by period, sport, or cup.

### bezeckyzavod.cz

- **Pagination:** `?strana=N` (verified). About 30 races per page in chronological order.
- Date format: `D.M.YYYY` (e.g. `2.7.2026`).
- Fields: name, city, distance (km), url. Notes/terrain often absent.
- To cover Jul–Oct, expect pages 1..~10.

### sport-base.cz

- Single static HTML page per year: `/kalendar/<year>.htm`. No pagination.
- Date format: `D.M.YYYY`.
- Fields: name, date, city. **Distance is not in the listing** — often blank; classify as `other` unless the name gives it away (OCR events, gladiator race, night run, etc.).
- Registration URLs point to external systems (sport-reg.cz, reg-sportvisio.cz, night-run.cz).

### runczech.com

- Single page: `/cs/akce?selected_year=YYYY`. No pagination.
- Race-series operator: only 4–5 races/year.
- Date format: `D. měsíc YYYY` (Czech long month, e.g. `5. září 2026`).
- Distance field lists multiple options — parse to emit one record per distance.

### run-tour.cz

- Single page. Race-series operator; ~8 races/year.
- Date format: `DD/MM/YYYY` (slash-separated).
- Race name is often just the city (`"Brno"`); distance always `10 km / 5 km / 3 km / štafety` (RunTour standard) — hardcode if not shown.
- URLs are relative (`/cs/kalendar/<city>`); prefix with `https://www.run-tour.cz`.

### behejlesy.cz

- Single page. Trail-series operator; ~8 races/year.
- Date format: `D. měsíce YYYY` (Czech long month).
- Distance not in the listing — Běhej lesy races are typically `12 km / 22 km / 35 km` (short / medium / long trail variants). If precise distance matters, fetch `/propozice/<location>` for the detail.
- Race name includes the venue (`"Běhej lesy Klínovec"`), so use the venue as `city`.

## After WebFetch — normalization checklist

- Parse `date_raw` → `date` (YYYY-MM-DD).
- Parse `distances_raw` → one race per distance; set `distance_km` numerically. Half-marathon = 21.0975, marathon = 42.195.
- Slugify `name` for the `id`: lowercase, strip diacritics (á→a, š→s, etc.), replace spaces with `-`, remove punctuation.
- Build `id`: `<source>-<date>-<slug>`. If multiple distances, append `-<distance>k` (e.g., `-42k`, `-21k`).
- `location`: split `location_raw` on comma; first token → `city`, second → `region` if present.

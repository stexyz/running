# Race data

Scraped race listings from Czech running-calendar sites. One JSON file per source. Refreshed by the `race-scrape` skill.

## Freshness

Files are refreshed by `race-scrape` when `fetched_at` is older than 7 days.

## File naming

`<source-id>-<year>.json` — per-source raw data (e.g., `ceskybeh-2026.json`, `top4running-2026.json`).
`all-<year>.json` — consolidated cross-source deduped view.

Source IDs:
- `behej` — JSON API, https://www.behej.com/terminovka
- `ceskybeh` — https://ceskybeh.cz/terminovka/
- `svetbehu` — https://www.svetbehu.cz/terminovka/
- `bezeckyzavod` — https://www.bezeckyzavod.cz/zavody/
- `sportbase` — https://www.sport-base.cz/kalendar/2026.htm
- `top4running` — https://top4running.cz/pg/kalendar-bezeckych-zavodu
- `runczech` — https://www.runczech.com/cs/akce (series operator)
- `runtour` — https://www.run-tour.cz/ (series operator)
- `behejlesy` — https://behejlesy.cz/ (series operator)

**Downstream skills (race-plan, race-render) should read `all-<year>.json`** unless they need source-specific fidelity.

## Schema

```json
{
  "source": "ceskybeh.cz",
  "fetched_at": "2026-07-01T10:23:00Z",
  "races": [
    {
      "id": "ceskybeh-2026-04-15-prazsky-maraton",
      "name": "Pražský maraton",
      "date": "2026-04-15",
      "location": {"city": "Praha", "region": "Praha", "country": "CZ"},
      "distance_km": 42.195,
      "type": "road-marathon",
      "url": "https://...",
      "source": "ceskybeh.cz"
    }
  ]
}
```

### Field notes

- `id`: `<source>-<date>-<name-slug>`, lowercase, hyphenated, unaccented.
- `date`: ISO 8601 date (YYYY-MM-DD).
- `location.region`: may be `null` if not extractable from the source.
- `location.country`: `CZ` for v1 sources.
- `distance_km`: numeric. For events with multiple distances, one race entry per distance (share the same event name; distinguish by `id` suffix and by distance).
- `type` enum: `road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`.
- `url`: canonical page for the race.

## Consolidated schema (`all-<year>.json`)

Same as above, except:
- `id`, `name`, `date`, `distance_km` are taken from the "most descriptive" contributor (longest name).
- `location` fields are picked independently as the first non-null value across contributors.
- `type` prefers any non-`other` classification.
- Adds a `sources` array (length ≥ 1) with per-contributor `{source, url, name}`. Singletons carry a one-element array so consumers can treat all records uniformly.

Deduplication rule: two entries merge when `date` matches, `distance_km` matches within ±0.5 km after rounding to 0.1, and normalized `city` (unaccented / lowercased) matches exactly. Entries with `city: null` never merge.

## Future sources (v2)

See `.claude/skills/race-scrape/references/foreign-sources.md` for researched candidates in DE / AT / SK / PL.

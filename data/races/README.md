# Race data

Scraped race listings from Czech running-calendar sites. One JSON file per source. Refreshed by the `race-scrape` skill.

## Freshness

Files are refreshed by `race-scrape` when `fetched_at` is older than 7 days.

## File naming

`<source-id>-<year>.json` — e.g., `ceskybeh-2026.json`, `top4running-2026.json`.

Source IDs (v1):
- `ceskybeh` — https://ceskybeh.cz/terminovka/
- `top4running` — https://top4running.cz/pg/kalendar-bezeckych-zavodu
- `svetbehu` — https://www.svetbehu.cz/terminovka/

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

## Future sources (v2)

See `~/.claude/skills/race-scrape/references/foreign-sources.md` for researched candidates in DE / AT / SK / PL.

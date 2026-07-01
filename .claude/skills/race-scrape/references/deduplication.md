# Cross-source deduplication

After per-source JSONs are written, produce a consolidated deduped file `data/races/all-<year>.json` combining all sources.

## Match rule

Two race entries are considered the same physical race when **all three** hold:

- `date` matches exactly (YYYY-MM-DD), AND
- `distance_km` values match within **±0.5 km** after rounding to 0.1 km (this catches marathon `42.2` vs `42.195` and similar rounding drift; it deliberately does NOT catch clearly different distances like `8.9` vs `9.6`), AND
- normalized `location.city` matches exactly. Normalization = strip diacritics, lowercase, collapse whitespace. `null` city → never merges (safe default).

Region and country are not used for matching. `Praha 4` vs `Praha 7` are treated as distinct cities (they are, effectively).

## Backfill step (must run before matching)

svetbehu stores a Czech kraj (region) in `location_raw` and leaves `city` `null`. Without backfilling, ~80 races never match anything and dedup catches nothing across sources.

For each race whose `city` is null:

1. Take its `date`. Gather all races from other sources on the same date.
2. Compute name similarity using `difflib.SequenceMatcher.ratio()` on normalized names (unaccented, lowercased, whitespace collapsed).
3. If the best match scores **≥ 0.6**, copy that source's `city` into this record. Keep this record's existing `region` and `country`.
4. If no match reaches the threshold, leave `city` null. The race will remain as a singleton in the consolidated file.

Backfilled cities are persisted back into the per-source JSON so subsequent runs don't repeat the work.

## Output shape (`all-<year>.json`)

```json
{
  "source": "consolidated",
  "fetched_at": "2026-07-01T22:56:20Z",
  "races": [
    {
      "id": "ceskybeh-2026-07-01-lachtan",
      "name": "Lachtan",
      "date": "2026-07-01",
      "location": {"city": "Praha 6", "region": null, "country": "CZ"},
      "distance_km": 6.2,
      "type": "trail",
      "sources": [
        {"source": "ceskybeh.cz", "url": "https://ceskybeh.cz/zavody/lachtan-2026/", "name": "Lachtan"},
        {"source": "svetbehu.cz", "url": "https://www.svetbehu.cz/terminovka/lachtanuv-memorial/", "name": "Lachtan"}
      ]
    }
  ]
}
```

Notes on merged records:
- `id`, `name`, `date`, `distance_km`: taken from the record with the longest `name` (proxy for "most descriptive"). Ties broken arbitrarily.
- `location`: first non-null value found across the group (city / region / country each picked independently).
- `type`: prefer any non-`other` classification; fall back to whatever the group has.
- `sources`: one entry per contributing source, preserving each source's own `url` and `name` variant. Length ≥ 1 always (singletons carry a one-element `sources` array so downstream code can treat all records uniformly).

## Why not use name similarity as the match rule directly?

Because name text varies too much across sources (translations, punctuation, added subtitles, e.g. `"Kolem Libotína"` vs `"Běh kolem Libotína"`). Name similarity is used only during the backfill step to find a plausible city; the actual merge decision still hinges on the strict `date + distance + city` triple, which produces no false positives worth worrying about.

## What this does NOT catch

- Cross-source records whose `city` cannot be backfilled (name too dissimilar or the other source doesn't have the race). These stay as singletons — acceptable, since the alternative (fuzzy-name merging) risks merging genuinely different races that share a date and distance.
- Records where sources disagree on distance beyond the ±0.5 km tolerance (e.g. ceskybeh says 9.6 km, svetbehu says 8.9 km for the same course). These stay separate and will surface in downstream views as duplicate-looking entries. Fix upstream by inspecting the source or accept it as a known data-quality artifact.

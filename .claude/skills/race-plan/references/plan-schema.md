# Plan file schema

Plans live at `plans/<year>-<goal-slug>.md`. YAML frontmatter + markdown body.

## Full frontmatter

```yaml
---
goal_race:
  name: string                    # e.g. "Berlin Marathon"
  date: YYYY-MM-DD                # required
  location: string                # e.g. "Berlin, DE"
  type: enum                      # see enum below
  priority: A                     # always A for the goal
build_ups:
  - name: string
    date: YYYY-MM-DD
    location: string
    type: enum
    priority: A | B | C           # C for train-through, B for important, A only if there's a second peak
    role: string                  # short description of why this race is here
    source: string | manual       # race id from data/races/*.json, or "manual" if user-entered
    warning: string (optional)    # inline warning if this race violates a rule
constraints:
  regions: [CZ, DE, AT, SK, PL]   # ISO country codes
  blackouts: [YYYY-MM-DD, ...]    # specific dates unavailable
  notes: string                   # free-form context
# Reserved for future TrainingPeaks / Strava integration — unused in v1
training_phase_by_week: {}
---
```

## Race type enum

Same as scraped data: `road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`.

## Priority enum

- `A` — peak race, taper mandatory (see `taper-rules.md`).
- `B` — important, soft taper (a few days to 1 week).
- `C` — train through, no taper.

## Validation

When reading a plan:

- All required fields present in `goal_race`.
- `date` fields are valid ISO dates.
- All `build_ups` dates fall before `goal_race.date`.
- No two build-ups on the same date.
- `type` in enum.
- `priority` in `A / B / C`.
- If validation fails, report the specific field and file name — don't silently proceed.

## Body

Below the frontmatter, free-form `# Plan notes` section. Not machine-read. User can jot training thoughts, rationale, gear reminders, etc.

## Slugifying goal race names for filenames

Lowercase, replace spaces with `-`, strip diacritics (á→a, š→s, ř→r, etc.), remove punctuation. Examples:

| Name                    | Slug                    |
|-------------------------|-------------------------|
| Berlin Marathon         | `berlin-marathon`       |
| Pražský půlmaraton      | `prazsky-pulmaraton`    |
| Běh okolo Brd           | `beh-okolo-brd`         |
| CCC (UTMB)              | `ccc-utmb`              |

Filename format: `<year-of-goal-race>-<slug>.md`, e.g., `2026-berlin-marathon.md`.

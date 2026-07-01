# Race Calendar Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build three composable Claude skills (`race-scrape`, `race-plan`, `race-render`) that let Štefan iteratively plan his race calendar around goal races, with build-ups and mandatory tapering, using scraped Czech race listings.

**Architecture:** Three markdown-based skills under `~/.claude/skills/`, communicating through JSON and markdown files stored in this repo (`data/races/`, `plans/`, `renders/`). No code — skills are instruction files for Claude, which uses WebFetch for scraping and file tools for reading/writing plans and renders.

**Tech Stack:** Markdown (SKILL.md files), JSON (race data), YAML frontmatter (plan files), HTML/CSS/JS (single-file HTML renders). No build system, no dependencies.

**Reference:** Spec at `docs/superpowers/specs/2026-07-01-race-calendar-skill-design.md`.

**File paths in this plan:**
- `SKILL_ROOT` refers to `/Users/stefan.pacinda@rossum.ai/.claude/skills/`
- `REPO_ROOT` refers to `/Users/stefan.pacinda@rossum.ai/dev/repository/github/running/`

---

## File structure

**Created in the repo (`REPO_ROOT`):**

```
data/races/README.md                            # schema documentation
plans/README.md                                 # what plans look like, how to name them
plans/archive/.gitkeep
renders/.gitkeep
.gitignore                                      # ignore generated renders (optional)
```

**Created under `SKILL_ROOT`:**

```
race-scrape/SKILL.md
race-scrape/references/source-notes.md
race-scrape/references/type-classification.md
race-scrape/references/foreign-sources.md
race-scrape/references/sample-race-json.md

race-plan/SKILL.md
race-plan/references/build-up-heuristics.md
race-plan/references/taper-rules.md
race-plan/references/plan-schema.md
race-plan/references/czech-regions.md

race-render/SKILL.md
race-render/references/markdown-view-a.md
race-render/references/markdown-view-b.md
race-render/references/html-template.html
```

Each SKILL.md is the main instruction file for Claude. Each `references/` file is loaded on demand when the skill needs specifics.

**Note on verification:** These are markdown-based skills — not code. Verification is not `pytest`. Each phase ends with a manual verification scenario: invoke the skill in a real Claude session and check the produced files match expectations. Commits happen at phase boundaries.

---

## Phase 0: Repo scaffolding

### Task 1: Create repo directory structure and data schema README

**Files:**
- Create: `REPO_ROOT/data/races/README.md`
- Create: `REPO_ROOT/plans/README.md`
- Create: `REPO_ROOT/plans/archive/.gitkeep`
- Create: `REPO_ROOT/renders/.gitkeep`

- [ ] **Step 1: Create the directory structure**

```bash
mkdir -p /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/data/races
mkdir -p /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/plans/archive
mkdir -p /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders
touch /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/plans/archive/.gitkeep
touch /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders/.gitkeep
```

- [ ] **Step 2: Write `data/races/README.md`**

Content:

````markdown
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
````

- [ ] **Step 3: Write `plans/README.md`**

Content:

````markdown
# Race plans

One file per goal race. Written and iterated on by the `race-plan` skill.

## Naming

`<year>-<goal-race-slug>.md` — e.g., `2026-berlin-marathon.md`. Lowercase, hyphenated.

Completed plans move to `archive/`.

## Structure

YAML frontmatter (machine-read by skills) + free-form markdown notes.

See the plan schema: `~/.claude/skills/race-plan/references/plan-schema.md`.

Minimal example:

```markdown
---
goal_race:
  name: Berlin Marathon
  date: 2026-09-27
  location: Berlin, DE
  type: road-marathon
  priority: A
build_ups: []
constraints:
  regions: [CZ, DE, AT, SK, PL]
  blackouts: []
  notes: ""
training_phase_by_week: {}
---

# Plan notes
```
````

- [ ] **Step 4: Verify structure**

Run:

```bash
ls -R /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/data /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/plans /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders
```

Expected: shows `data/races/README.md`, `plans/README.md`, `plans/archive/.gitkeep`, `renders/.gitkeep`.

- [ ] **Step 5: Commit**

```bash
git -C /Users/stefan.pacinda@rossum.ai/dev/repository/github/running add data/races/README.md plans/README.md plans/archive/.gitkeep renders/.gitkeep
git -C /Users/stefan.pacinda@rossum.ai/dev/repository/github/running commit -m "chore: scaffold data/, plans/, renders/ directories"
```

---

## Phase 1: `race-scrape` skill

### Task 2: Create `race-scrape/SKILL.md`

**Files:**
- Create: `SKILL_ROOT/race-scrape/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/references
```

- [ ] **Step 2: Write `SKILL.md`**

Content (write this exact file — the `description` field must include trigger phrases so Claude auto-invokes):

````markdown
---
name: race-scrape
description: Use when the user wants to refresh, scrape, or update race calendar data from Czech running sites (top4running.cz, ceskybeh.cz, svetbehu.cz), or when another skill needs fresh race data. Fetches, normalizes, and writes JSON files to data/races/.
---

# race-scrape

Fetches race listings from three Czech running-calendar sites, normalizes them into a shared schema, and writes one JSON file per source to `data/races/` in the current repo.

**Repo assumption:** the current working directory is a running-planning repo with a `data/races/` directory. If not, ask the user where to write.

## Sources (v1)

| Source ID     | URL                                                    |
|---------------|--------------------------------------------------------|
| `ceskybeh`    | https://ceskybeh.cz/terminovka/                        |
| `top4running` | https://top4running.cz/pg/kalendar-bezeckych-zavodu    |
| `svetbehu`    | https://www.svetbehu.cz/terminovka/                    |

Output files: `data/races/<source-id>-<year>.json`.

## Freshness threshold

**7 days.** For each source, check `fetched_at` in the existing JSON file. If missing or older than 7 days, refresh.

## Execution steps

1. **Check freshness.** For each source ID, look at `data/races/<source>-<year>.json`. If missing or `fetched_at` more than 7 days ago, mark for refresh. If all sources are fresh, report `"All sources fresh, nothing to do."` and stop.

2. **Confirm scope.** Report to the user which sources are being refreshed and why (e.g., "Refreshing ceskybeh: last fetched 12 days ago").

3. **Fetch each stale source with WebFetch.** For each source, use `WebFetch` on its URL with a prompt asking for structured extraction — see `references/source-notes.md` for the per-site prompt template.

4. **Normalize each race** into the shared schema (see `data/races/README.md` in the repo). Specifically:
   - Parse Czech dates. Common formats: `15. dubna 2026`, `15.4.2026`, `15. 4. 2026`. See `references/source-notes.md` for month name mapping.
   - Classify `type` using `references/type-classification.md`.
   - Build `id` as `<source>-<YYYY-MM-DD>-<name-slug>` (lowercase, hyphenated, unaccented Czech chars).
   - Extract `city` and (if available) `region`. Leave `region: null` if unclear.
   - `country`: always `CZ` for v1 sources.

5. **Handle events with multiple distances.** If one calendar entry lists e.g. "5k / 10k / HM", emit one race entry per distance. Same date, same location, distinct `id` (suffix with distance) and distinct `type`.

6. **Write JSON.** One file per source, containing `source`, `fetched_at` (current UTC ISO 8601), and `races` array. Sort races by `date` ascending.

7. **Report summary.** `"Refreshed <N> sources. Total races: <total>. New since last fetch: <delta>."`

## Failure handling

- If a source fails (network error, layout change causing WebFetch to return nothing usable): **keep the previous JSON in place** — do not overwrite with empty data. Log the error, continue with the other sources.
- If all sources fail: warn the user and stop.
- If WebFetch returns clearly malformed data (no dates parseable), do not write partial data — treat as failure for that source.

## Loading references

- `references/source-notes.md` — per-site WebFetch prompt templates, quirks, date format map.
- `references/type-classification.md` — classification heuristics (name keywords, distance ranges).
- `references/foreign-sources.md` — v2 candidates only. Do not use in v1.
- `references/sample-race-json.md` — canonical example of the output shape.

## How to verify this worked

1. `ls data/races/` shows one JSON file per successfully refreshed source.
2. Each JSON file parses and has non-empty `races` array.
3. `fetched_at` on refreshed files is within the last few minutes.
4. Spot-check three races: date parseable, `type` in the allowed enum, `id` matches format.
5. Report summary matches the actual counts in the files.
````

- [ ] **Step 3: Verify the file was written**

Run:

```bash
ls -la /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/SKILL.md
head -5 /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/SKILL.md
```

Expected: file exists, first lines show frontmatter with `name: race-scrape`.

- [ ] **Step 4: Commit** (skills live outside the repo, so no commit needed — noted here so the executing agent doesn't try)

Skip commit; `~/.claude/skills/` is not a git repo. Move on.

---

### Task 3: Create `references/source-notes.md`

**Files:**
- Create: `SKILL_ROOT/race-scrape/references/source-notes.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
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

Also handle: `15.4.2026`, `15. 4. 2026`, `15/4/2026`, `2026-04-15`.

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

### top4running.cz

- May have advertising / commercial races mixed in.
- Distances sometimes only in the detail page, not the listing. If missing from listing, set `distance_km: null` and infer `type: other`.

### svetbehu.cz

- Structure similar to ceskybeh but with more editorial descriptions.
- Some entries are "series" (multi-race). Emit each round as a separate race entry using the round date.

## After WebFetch — normalization checklist

- Parse `date_raw` → `date` (YYYY-MM-DD).
- Parse `distances_raw` → one race per distance; set `distance_km` numerically. Half-marathon = 21.0975, marathon = 42.195.
- Slugify `name` for the `id`: lowercase, strip diacritics (á→a, š→s, etc.), replace spaces with `-`, remove punctuation.
- Build `id`: `<source>-<date>-<slug>`. If multiple distances, append `-<distance>k` (e.g., `-42k`, `-21k`).
- `location`: split `location_raw` on comma; first token → `city`, second → `region` if present.
````

- [ ] **Step 2: Verify**

Run:

```bash
wc -l /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/references/source-notes.md
```

Expected: file has ~60+ lines.

---

### Task 4: Create `references/type-classification.md`

**Files:**
- Create: `SKILL_ROOT/race-scrape/references/type-classification.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Race type classification

Heuristics to assign a `type` value from name, distance, and notes.

## Allowed values

`road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`

## Priority order for classification

Evaluate top to bottom. First match wins.

### 1. Ultra

- Distance ≥ 43 km, OR
- Name contains: `ultra`, `100`, `50k`, `50 km`

→ `ultra`

### 2. Vertical

- Name contains: `vertical`, `vk`, `výběh`, `do vrchu`

→ `vertical`

### 3. Duathlon

- Name contains: `duatlon`, `duathlon`

→ `duathlon`

### 4. Night race

- Name contains: `noční`, `night`, `nocturne`

→ `night-race`

(Overrides road distance classifications — a "noční půlmaraton" is `night-race`, not `road-half`.)

### 5. Trail

- Notes indicate trail surface, OR
- Name contains: `trail`, `trailový`, `horský`, `krosový`, `terénní`, `lesní`

→ `trail`

### 6. Road marathon

- Distance between 41.5 and 43 km, AND not trail/ultra/night above

→ `road-marathon`

### 7. Road half-marathon

- Distance between 20.5 and 21.5 km, AND not trail/ultra/night above

→ `road-half`

### 8. Road 10k

- Distance between 9.5 and 10.5 km, AND not trail/night above

→ `road-10k`

### 9. Road 5k

- Distance between 4.5 and 5.5 km, AND not trail/night above

→ `road-5k`

### 10. Fallback

- Anything else → `other`

## Notes

- If `distance_km` is `null`, only name-based rules can fire; anything unclassified → `other`.
- The classification is heuristic. Users can correct race types in plan files manually.
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/references/type-classification.md`

Expected: file exists.

---

### Task 5: Create `references/foreign-sources.md`

**Files:**
- Create: `SKILL_ROOT/race-scrape/references/foreign-sources.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Foreign race sources — v2 research

**Not implemented in v1.** These are candidate sources for expanding scraping to DE / AT / SK / PL races that are easily reachable from CZ (good public transport or highway connection).

## Germany (DE)

- **laufkalender.de** — comprehensive German-language running calendar, filterable by region and date. Coverage: whole DE. Strong candidate.
- **runme.de** — event listings, more marathon-focused.

## Austria (AT)

- **runaustria.at** — Austrian running events, well-structured.
- **laufkalender.at** — analogous to the DE site.

## Slovakia (SK)

- **beh.sk** — Slovak equivalent of ceskybeh.cz. Coverage: whole SK.
- **behamesa.sk** — additional listing.

## Poland (PL)

- **maratonypolskie.pl** — Polish marathon and long-distance calendar.
- **enduhub.com** — international, includes PL events with results.

## Integration approach (v2)

Each source needs:
1. Entry in `SKILL.md` sources table.
2. Per-site prompt template in `source-notes.md`.
3. Date format map if not already covered.
4. Confirmation that WebFetch can parse it (some sites gate content behind JS — spot-check first).

Shared race schema does not need changes — `country` field accommodates all four codes.

## Travel-worthiness heuristic (v2)

Not a scraping concern, but relevant for `race-plan`: a foreign race is "reachable" if it's within ~4 hours by direct public transport or highway from Prague / Brno. `race-plan` should let the user filter by this in the constraints phase.
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/references/foreign-sources.md`

Expected: file exists.

---

### Task 6: Create `references/sample-race-json.md`

**Files:**
- Create: `SKILL_ROOT/race-scrape/references/sample-race-json.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Sample race JSON

Canonical shape for a scraped race-list file. Use as a reference when writing output.

```json
{
  "source": "ceskybeh.cz",
  "fetched_at": "2026-07-01T10:23:00Z",
  "races": [
    {
      "id": "ceskybeh-2026-04-05-prazsky-pulmaraton-21k",
      "name": "Pražský půlmaraton",
      "date": "2026-04-05",
      "location": {"city": "Praha", "region": "Praha", "country": "CZ"},
      "distance_km": 21.0975,
      "type": "road-half",
      "url": "https://ceskybeh.cz/zavod/prazsky-pulmaraton",
      "source": "ceskybeh.cz"
    },
    {
      "id": "ceskybeh-2026-05-17-krosovy-behokolo-brd-10k",
      "name": "Krosový běh okolo Brd",
      "date": "2026-05-17",
      "location": {"city": "Příbram", "region": "Středočeský", "country": "CZ"},
      "distance_km": 10.0,
      "type": "trail",
      "url": "https://ceskybeh.cz/zavod/krosovy-beh-brd",
      "source": "ceskybeh.cz"
    },
    {
      "id": "ceskybeh-2026-06-21-nocni-desitka-brno-10k",
      "name": "Noční desítka Brno",
      "date": "2026-06-21",
      "location": {"city": "Brno", "region": "Jihomoravský", "country": "CZ"},
      "distance_km": 10.0,
      "type": "night-race",
      "url": "https://ceskybeh.cz/zavod/nocni-desitka-brno",
      "source": "ceskybeh.cz"
    }
  ]
}
```

## Notes on the examples

- Third entry demonstrates `night-race` overriding what would otherwise be `road-10k`.
- Second entry demonstrates trail classification from name keyword.
- `id` includes distance suffix when the event has multiple distances (see first entry — `-21k`).
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-scrape/references/`

Expected: shows all four reference files (`source-notes.md`, `type-classification.md`, `foreign-sources.md`, `sample-race-json.md`).

---

### Task 7: Phase 1 verification scenario

**Purpose:** confirm `race-scrape` actually works end-to-end by running it against real sites.

- [ ] **Step 1: In a fresh Claude Code session (cwd = `REPO_ROOT`), invoke:**

```
Please refresh my race calendar data.
```

Expected behavior: Claude invokes the `race-scrape` skill, reports that no data exists yet, calls WebFetch on all three sources, writes three JSON files to `data/races/`, reports counts.

- [ ] **Step 2: Verify output files**

Run:

```bash
ls -la /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/data/races/
```

Expected: three JSON files present, all created within the last few minutes.

- [ ] **Step 3: Spot-check contents**

For each file, verify:
- Parses as JSON (try: `python3 -c "import json; d=json.load(open('data/races/ceskybeh-2026.json')); print(len(d['races']), 'races')"`).
- Has non-empty `races` array.
- `fetched_at` is recent.
- Sample 3 races: `date` matches YYYY-MM-DD, `type` in allowed enum, `id` follows format.

- [ ] **Step 4: If any source failed, investigate**

Common causes: site layout changed (update `source-notes.md` prompt), rate limiting (retry), JS-gated content (may need alternate approach — flag in `foreign-sources.md`-style note).

- [ ] **Step 5: Commit the scraped data**

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
git add data/races/*.json
git commit -m "feat: initial race data scrape from Czech sources"
```

---

## Phase 2: `race-plan` skill

### Task 8: Create `race-plan/SKILL.md`

**Files:**
- Create: `SKILL_ROOT/race-plan/SKILL.md`

- [ ] **Step 1: Create skill directory**

```bash
mkdir -p /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/references
```

- [ ] **Step 2: Write `SKILL.md`**

Content:

````markdown
---
name: race-plan
description: Use when the user wants to plan a race calendar, plan build-up races around a goal race, or iterate on an existing race plan. Interactive session that reads scraped race data, applies build-up and taper heuristics, and writes a plan file to plans/.
---

# race-plan

Interactive skill for planning a race season around one or more goal races. Reads scraped race data from `data/races/`, applies build-up and mandatory taper rules, and writes a plan to `plans/<year>-<goal-slug>.md`.

**Repo assumption:** current working directory has `data/races/` and `plans/` directories. If not, ask the user.

## Session flow

### Phase 1 — Identify context

1. List `plans/*.md` (skip `plans/archive/`).
2. Ask: `"Iterate on an existing plan, or start a new one?"`
   - If existing: load its frontmatter, ask `"What would you like to change?"`, then jump to Phase 4 in modify mode.
   - If new: proceed to Phase 2.

### Phase 2 — Gather goal race (new plans only)

Ask one question at a time:

1. **Goal race name.** If free text, search `data/races/*.json` for fuzzy matches on `name`. If matches, present numbered candidates and ask to confirm or say "manual entry".
2. **Goal race date.** Pre-fill if picked from data.
3. **Goal race location.** Pre-fill if picked from data. Format: `City, Country`.
4. **Race type.** Pre-fill from data if available. Otherwise ask, presenting the allowed enum from `references/plan-schema.md`.
5. **Priority level.** A / B / C. Explain briefly: A = peak for this, B = important but not peak, C = train through.

### Phase 3 — Gather constraints

Ask one at a time:

1. **Regions willing to travel to.** Default suggestion: `"CZ (any region), plus DE / AT / SK / PL if good public transport / highway. OK?"` Take free text, save as list of country codes plus notes.
2. **Blackout dates.** Ask for specific dates or weekends unavailable. Empty list is fine.
3. **Number of build-up races.** Ask: `"How many build-ups? (rough number, or say 'let me suggest')"`. Default for marathon block: 3–5.
4. **Current fitness sanity check.** Free text. No integration in v1; use as context for build-up suggestions.

### Phase 4 — Propose build-ups

1. **Freshness check.** If any file in `data/races/` has `fetched_at` older than 7 days or is missing, tell the user and offer to invoke the `race-scrape` skill first.

2. **Filter candidate races.** From all `data/races/*.json`, filter to races that:
   - Fall between "today" and the goal race date.
   - Match one of the regions the user is willing to travel to.
   - Are not on blackout dates.
   - Match acceptable build-up types for the goal type (see `references/build-up-heuristics.md`).

3. **Apply taper rules.** Load `references/taper-rules.md`. Compute the pre-A-race taper window from the goal race backwards. Exclude any candidate race inside the taper window from build-up suggestions (though the plan can still contain them — see step 6).

4. **Apply spacing rules.** Between hard candidate races (any A/B), enforce minimum ~2 weeks. Warn if the user's manual picks violate this.

5. **Propose 2–3 candidate calendars.** For each, list the picks with dates, distances, and a one-line rationale per race (e.g., `"HM in Karlovy Vary 5 weeks out — pacing rehearsal, easy train from Prague"`). Ask the user to pick or ask for alternatives.

6. **Handle manual additions.** If the user adds a race that violates taper or spacing rules, warn loudly but allow it — record it with an inline warning comment.

### Phase 5 — Write the plan

1. Generate filename: `<year>-<goal-slug>.md` where `<goal-slug>` is the goal race name slugified (lowercase, hyphens, no diacritics).
2. Build the frontmatter per `references/plan-schema.md`.
3. Write the file. If a file with that name exists (iteration mode), overwrite it — but back up the previous version as `plans/archive/<name>.<timestamp>.md.bak` first.
4. Include an empty `# Plan notes` section below the frontmatter.

### Phase 6 — Offer to render

Ask: `"Plan written to plans/<name>.md. Want me to render it? Options: markdown detailed timeline, markdown proportional week grid, HTML, or all three."`

If yes, invoke the `race-render` skill.

## Loading references

- `references/plan-schema.md` — frontmatter shape and validation rules.
- `references/build-up-heuristics.md` — per-goal-type build-up patterns.
- `references/taper-rules.md` — the mandatory taper table.
- `references/czech-regions.md` — region names and travel-time hints.

## How to verify this worked

1. `plans/<name>.md` exists with valid YAML frontmatter.
2. `goal_race`, `build_ups`, and `constraints` blocks are populated.
3. No build-up race violates the taper rules (unless the user explicitly overrode it, in which case the plan contains a warning comment).
4. Every build-up race exists in `data/races/*.json` OR is marked as `source: manual` in the plan.
5. Dates are before the goal race date.
````

- [ ] **Step 3: Verify**

Run: `head -5 /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/SKILL.md`

Expected: shows frontmatter with `name: race-plan`.

---

### Task 9: Create `references/plan-schema.md`

**Files:**
- Create: `SKILL_ROOT/race-plan/references/plan-schema.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
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
training_phase_by_week: {}        # reserved for future TrainingPeaks/Strava integration; empty in v1
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
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/references/plan-schema.md`

Expected: file exists.

---

### Task 10: Create `references/taper-rules.md`

**Files:**
- Create: `SKILL_ROOT/race-plan/references/taper-rules.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Taper rules

Applied by `race-plan` when placing build-ups and when validating a plan.

## Pre-A-race taper — mandatory

No hard races (A or B) inside the taper window before an A-race. C-races are allowed if the user explicitly overrides.

| Goal race type          | Taper window (weeks) |
|-------------------------|---------------------|
| road-marathon           | 3                   |
| ultra                   | 3                   |
| road-half               | 2                   |
| trail (long / mountain) | 2–3 (ask user)      |
| road-10k                | ~10 days (~1.5 wk)  |
| road-5k                 | 1                   |
| night-race              | match by distance   |
| duathlon                | 1–2 (ask user)      |
| vertical                | 1                   |

**How to compute:** taper start date = `goal_race.date - taper_weeks * 7 days`. Any race with `date >= taper_start` and `date < goal_race.date` is inside the taper window.

## Pre-B-race taper — soft

Not enforced. Warn if:
- A B-race is a road-marathon or road-half and there's another hard race within 7 days before it.
- A B-race is shorter (10k / 5k) and there's another hard race within 3 days before it.

## Minimum spacing between hard races

At least 2 weeks (14 days) between any two A/B races. Warn on closer spacing, but allow.

## Post-race recovery guidance

Not enforced (out of scope for v1 — could be a v2 rule for TrainingPeaks integration). Note for the user in the "Plan notes" section if you notice a very tight sequence.

## Reporting violations

When a proposed or existing plan violates a hard rule:

- In interactive planning (`race-plan` Phase 4): don't offer the violating candidate; explain why.
- In rendering (`race-render`): surface as a top-level `⚠ Warnings` block.
- In modifying: warn but allow, and add an inline `warning:` field to the offending build-up entry in the plan frontmatter.
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/references/taper-rules.md`

Expected: file exists.

---

### Task 11: Create `references/build-up-heuristics.md`

**Files:**
- Create: `SKILL_ROOT/race-plan/references/build-up-heuristics.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Build-up heuristics

Suggested build-up patterns per goal race type. These are starting points — the user can add, remove, or shift races.

Weeks are measured backwards from goal race date.

## Goal: road-marathon

| Weeks out | Type         | Priority | Purpose                                |
|-----------|--------------|----------|----------------------------------------|
| 8–10      | road-10k     | B        | Threshold pacing, calibrate current fitness |
| 4–6       | road-half    | B        | Marathon pace / faster than MP rehearsal, kit test |
| 2–3       | road-5k or road-10k | C  | Optional short tune-up, easy effort   |

Taper: 3 weeks. No hard races in weeks 0–3 before goal.

## Goal: ultra

| Weeks out | Type              | Priority | Purpose                                    |
|-----------|-------------------|----------|--------------------------------------------|
| 6–10      | trail (medium)    | B        | Terrain rehearsal, nutrition test          |
| 4–6       | trail (long)      | B or C   | Big training run with support (often as C) |
| Earlier   | vertical (optional) | C      | Climbing legs, if goal has significant vert |

Taper: 3 weeks. No hard races in weeks 0–3 before goal.

## Goal: trail (medium/long)

Similar to ultra but scaled down. HM or trail-HM ~4–6 weeks out, C-race trail 8–10 weeks out.

Taper: 2–3 weeks (ask user based on distance).

## Goal: road-half

| Weeks out | Type         | Priority | Purpose                             |
|-----------|--------------|----------|-------------------------------------|
| 6–8       | road-10k     | B        | Threshold, pacing calibration       |
| 3–4       | road-5k      | C        | Short race pace exposure            |

Taper: 2 weeks.

## Goal: road-10k

| Weeks out | Type         | Priority | Purpose                              |
|-----------|--------------|----------|--------------------------------------|
| 4–5       | road-5k or road-10k | B | Pacing / speed race                  |
| ~2        | road-5k or parkrun  | C | Time trial / sharpener, easy effort  |

Taper: ~10 days.

## Goal: road-5k

| Weeks out | Type       | Priority | Purpose             |
|-----------|------------|----------|---------------------|
| 4         | road-5k    | B or C   | Race-pace exposure  |
| ~2        | parkrun    | C        | Sharpener           |

Taper: 1 week.

## Spacing rule (applies to all goals)

Minimum 2 weeks between any two hard races (A or B). If a candidate build-up is closer, either drop it, downgrade to C, or push to another date.

## Fitness caveats

If the user reports coming back from injury or a training gap:
- Skip early build-ups (>10 weeks out).
- Downgrade B → C to reduce race-recovery cost.
- Prefer shorter distances in build-ups.

If the user reports coming off a strong recent block:
- Standard patterns apply.
- Consider adding one more early build-up if they want more racing.
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/references/build-up-heuristics.md`

Expected: file exists.

---

### Task 12: Create `references/czech-regions.md`

**Files:**
- Create: `SKILL_ROOT/race-plan/references/czech-regions.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Czech regions and travel-time hints

Rough guide for filtering races by user's willingness to travel. All times are one-way by car, ballpark.

## Regions (kraj)

| Region (Czech)     | Region (English)          | Notes                              |
|--------------------|---------------------------|------------------------------------|
| Praha              | Prague                    | Capital, most races                |
| Středočeský        | Central Bohemia           | Surrounds Prague                   |
| Jihočeský          | South Bohemia             | České Budějovice                   |
| Plzeňský           | Pilsen                    | Plzeň                              |
| Karlovarský        | Karlovy Vary              | Spa area, western CZ               |
| Ústecký            | Ústí nad Labem            | North, near DE border              |
| Liberecký          | Liberec                   | North, near DE and PL borders      |
| Královéhradecký    | Hradec Králové            | North-east                         |
| Pardubický         | Pardubice                 | East-central                       |
| Vysočina           | Vysočina (highlands)      | Between Bohemia and Moravia        |
| Jihomoravský       | South Moravia             | Brno, near AT and SK borders       |
| Olomoucký          | Olomouc                   | Central Moravia                    |
| Zlínský            | Zlín                      | East Moravia, near SK border       |
| Moravskoslezský    | Moravia-Silesia           | Ostrava, near PL and SK borders    |

## Travel time from Prague (approximate)

| Region                  | Car (h) | Train (h)   |
|-------------------------|---------|-------------|
| Praha / Středočeský     | 0–1     | 0–1         |
| Plzeňský                | 1       | 1.5         |
| Ústecký / Liberecký     | 1–1.5   | 1.5–2       |
| Karlovarský             | 2       | 3           |
| Jihočeský               | 2       | 2.5–3       |
| Vysočina                | 2       | 2.5         |
| Královéhradecký / Pardubický | 1.5–2 | 1.5–2   |
| Jihomoravský (Brno)     | 2–2.5   | 2.5         |
| Olomoucký               | 3       | 3           |
| Zlínský / Moravskoslezský | 3.5–4 | 3.5–4       |

## Cross-border reachable from Prague / Brno

Reasonable if `race-plan`'s constraint step includes DE / AT / SK / PL:

| Destination           | From Prague     | From Brno       |
|-----------------------|-----------------|-----------------|
| Dresden (DE)          | 2 h car / train | —               |
| Berlin (DE)           | 4–5 h train     | —               |
| Nuremberg (DE)        | 3.5 h car       | —               |
| Vienna (AT)           | 3.5–4 h train   | 1.5 h train     |
| Bratislava (SK)       | 4 h train       | 1.5 h train     |
| Kraków (PL)           | 5–6 h train     | 3.5 h car       |
| Wrocław (PL)          | 4–5 h car       | 3 h car         |

## Usage in race-plan

- When filtering by region, match `race.location.region` (Czech name) against the user's stated regions.
- For foreign races (`country != "CZ"`), always ask the user to confirm — v1 doesn't scrape foreign races, so any foreign entry is manual and the user already picked it.
- Use travel-time tables to add rationale when suggesting/deprioritizing races (e.g., `"Race in Ostrava is 4 h from Prague — heavy travel day, factor in recovery"`).
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-plan/references/`

Expected: shows all four reference files.

---

### Task 13: Phase 2 verification scenario

**Purpose:** confirm `race-plan` produces a valid plan.

- [ ] **Step 1: In a fresh Claude Code session (cwd = `REPO_ROOT`), invoke:**

```
Let's plan my race calendar. Goal: something in the fall.
```

Expected: Claude invokes `race-plan`, checks `plans/`, sees no existing plans, walks through Phase 2 questions.

Provide reasonable answers:
- Goal race: pick something visible in the scraped data (e.g., a fall marathon).
- Priority: A.
- Regions: CZ, plus DE if easy.
- Blackouts: none.
- Build-ups: let Claude suggest.
- Fitness: "coming off a good spring block, no injuries".

- [ ] **Step 2: Confirm proposal quality**

Claude should propose 2–3 build-up calendars, each with:
- Names of real races from `data/races/*.json`.
- Dates before the goal race.
- No race inside the taper window.
- Rationale per race.

- [ ] **Step 3: Pick one, let Claude write the plan**

- [ ] **Step 4: Verify the plan file**

Run:

```bash
ls /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/plans/
```

Expected: one `.md` file matching `<year>-<slug>.md`.

Manually inspect:
- Frontmatter parses as YAML.
- `goal_race` block complete.
- `build_ups` block has entries.
- Every `build_ups[].date < goal_race.date`.
- No build-up inside the taper window (unless flagged).

- [ ] **Step 5: Test iteration mode**

In a new session, invoke:

```
Let's iterate on my [race name] plan.
```

Expected: Claude finds the plan, asks what to change, applies the change, backs up the old version to `plans/archive/`.

- [ ] **Step 6: Commit the plan**

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
git add plans/*.md plans/archive/*.md.bak 2>/dev/null
git commit -m "feat: initial race plan"
```

---

## Phase 3: `race-render` skill

### Task 14: Create `race-render/SKILL.md`

**Files:**
- Create: `SKILL_ROOT/race-render/SKILL.md`

- [ ] **Step 1: Create skill directory**

```bash
mkdir -p /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references
```

- [ ] **Step 2: Write `SKILL.md`**

Content:

````markdown
---
name: race-render
description: Use when the user wants to render, visualize, or view a race plan. Reads a plan file from plans/, produces a markdown timeline (two view options), or a self-contained HTML calendar page written to renders/.
---

# race-render

Takes a plan file from `plans/` and produces a visual representation. Two markdown views (for iteration) and one HTML view (for polished output).

**Repo assumption:** cwd has `plans/` and `renders/` directories.

## Trigger and inputs

When invoked, ask the user:

1. **Which plan?** List `plans/*.md` (skip `archive/`). If one exists, default to it. If multiple, ask.
2. **Which output?** Options:
   - `markdown-detailed` — vertical timeline with weeks-between annotations.
   - `markdown-weekly` — proportional week grid, one line per calendar week.
   - `html` — self-contained HTML page.
   - `all` — produce all three.

## Execution

1. Load and validate the plan file per `~/.claude/skills/race-plan/references/plan-schema.md`.
2. Compute derived data:
   - Sort all races (goal + build-ups) by date.
   - Compute gaps between consecutive races in weeks (rounded to 1 decimal if fractional).
   - Compute the pre-A-race taper window from `~/.claude/skills/race-plan/references/taper-rules.md`.
   - Identify rule violations (any race inside taper window, spacing under 2 weeks between hard races).
3. Render per the requested view(s) using the corresponding reference template.
4. Write outputs:
   - Markdown views → print to conversation directly (user is iterating).
   - HTML view → write to `renders/<plan-slug>.html`, then tell the user the path.
5. If violations exist, always emit a `⚠ Warnings` block, regardless of view.

## Loading references

- `references/markdown-view-a.md` — detailed timeline format and template.
- `references/markdown-view-b.md` — proportional week grid format and template.
- `references/html-template.html` — base HTML with placeholder tokens.

## How to verify this worked

1. For `markdown-detailed`: output contains one block per race, weeks-between annotations between them, taper-window annotation before the A-race, and (if applicable) a warnings block.
2. For `markdown-weekly`: output has one line per calendar week from plan start to goal race, with `██` markers on race weeks and `░░` on taper weeks.
3. For `html`: `renders/<plan-slug>.html` exists, opens in a browser, shows header + timeline strip + month grid + race table. No missing tokens (grep for `{{` or `%%` should return nothing).
4. Warnings block accurately lists all violations (or is absent if none).
````

- [ ] **Step 3: Verify**

Run: `head -5 /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/SKILL.md`

Expected: shows frontmatter.

---

### Task 15: Create `references/markdown-view-a.md`

**Files:**
- Create: `SKILL_ROOT/race-render/references/markdown-view-a.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Markdown View A — detailed timeline

## Format

```
<Year> — <Goal race name> Block

<optional table view — see below>

  <Date> (Day)  ┃ P ┃ <Race name>                          <Type · Location>
                │       role: <role text>
                │
                │  ↕ <N> weeks
                │
  <Date> (Day)  ┃ P ┃ <Race name>                          <Type · Location>
                │       role: <role text>
                │
                │  ↕ <N> weeks   ← <optional annotation, e.g. taper start>
                │
  <Date> (Day)  ┃ A ┃ <Goal race name>                     <Type · Location>  ★

<optional ⚠ Warnings block>
```

## Rendering rules

- Dates: `Apr 05 (Sun)` — three-letter month, zero-padded day, three-letter day-of-week in parens.
- `P` is priority — one of `A`, `B`, `C`. Padded to 1 char.
- Race name: as-is from the plan.
- Type: friendly label from enum:
  - `road-marathon` → `Marathon`
  - `road-half` → `HM`
  - `road-10k` → `10k`
  - `road-5k` → `5k`
  - `trail` → `Trail`
  - `ultra` → `Ultra`
  - `vertical` → `Vertical`
  - `duathlon` → `Duathlon`
  - `night-race` → `Night`
  - `other` → `Other`
- Location: `City, Country` — country code omitted if `CZ`.
- Weeks-between: computed as `(next.date - prev.date).days / 7`, rounded to 0.5. Show as integer when whole, otherwise like `4.5`.
- Taper annotation: when the gap crosses into the goal's taper window, append `← <N>-week A-race taper starts <date>` to the gap line.
- Goal race: mark with `★` at end of line.

## Compact table (prepend before the timeline)

```
| Date       | P | Name                      | Type   | Location        |
|------------|---|---------------------------|--------|-----------------|
| 2026-04-05 | B | Pražský půlmaraton        | HM     | Praha           |
| 2026-08-16 | C | Běh okolo Brd             | Trail  | Příbram         |
| 2026-09-27 | A | Berlin Marathon           | Marathon | Berlin, DE    |
```

## Warnings block (when applicable)

```
⚠ Warnings
  • Race on <date> falls inside the <N>-week A-race taper window
  • Only <M> days between build-up on <date> and follow-up on <date>
```

Omit the block entirely if no violations.

## Full worked example

```
2026 — Berlin Marathon Block

| Date       | P | Name                | Type     | Location       |
|------------|---|---------------------|----------|----------------|
| 2026-04-05 | B | Pražský půlmaraton  | HM       | Praha          |
| 2026-08-16 | C | Běh okolo Brd       | Trail    | Příbram        |
| 2026-09-27 | A | Berlin Marathon     | Marathon | Berlin, DE     |

  Apr 05 (Sun)  ┃ B ┃ Pražský půlmaraton                    HM · Praha
                │       role: pacing rehearsal
                │
                │  ↕ 19 weeks
                │
  Aug 16 (Sun)  ┃ C ┃ Běh okolo Brd                          Trail · Příbram
                │       role: long training run with support
                │
                │  ↕ 6 weeks   ← 3-week A-race taper starts Sep 06
                │
  Sep 27 (Sun)  ┃ A ┃ Berlin Marathon                        Marathon · Berlin, DE  ★
```
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references/markdown-view-a.md`

Expected: file exists.

---

### Task 16: Create `references/markdown-view-b.md`

**Files:**
- Create: `SKILL_ROOT/race-render/references/markdown-view-b.md`

- [ ] **Step 1: Write the file**

Content:

````markdown
# Markdown View B — proportional week grid

One line per calendar week from plan start to goal race, including idle weeks, so visual distance reflects real time.

## Format

```
<Year> — <Goal race name> Block  (proportional week grid)

W-<n> (<Mon-date>)  <marker>  <optional race info>
W-<n> (<Mon-date>)  ·
W-<n> (<Mon-date>)  ·
...
W-00 (<Mon-date>)  ██ A  <Goal race name> ★
```

## Rendering rules

- **Weeks-out** counted backwards from goal race. `W-00` is the goal week (the Monday of that ISO week).
- **Plan start:** the Monday of the ISO week containing the earliest build-up race. Idle weeks between plan start and first race are still shown (they surface any "long dry stretch" in the calendar).
- **Line format:** `W-<XX> (<YYYY-MM-DD>)  <marker>  <race summary>`
- **Markers:**
  - `██ A` / `██ B` / `██ C` for race weeks.
  - `░░` for taper weeks.
  - `·` for idle weeks.
  - If both taper and race apply to the same week: use `██` with the race priority, and add ` [inside taper]` to the race summary.
- **Race summary:** `<Name> (<Type> · <City>)`
- **Goal week:** always ends with `★`.

## Warnings block

Same as View A. Prepend after the header.

## Full worked example

```
2026 — Berlin Marathon Block  (proportional week grid)

W-25 (2026-03-30)  ██ B  Pražský půlmaraton (HM · Praha, race Sun 04-05)
W-24 (2026-04-06)  ·
W-23 (2026-04-13)  ·
W-22 (2026-04-20)  ·
W-21 (2026-04-27)  ·
W-20 (2026-05-04)  ·
W-19 (2026-05-11)  ·
W-18 (2026-05-18)  ·
W-17 (2026-05-25)  ·
W-16 (2026-06-01)  ·
W-15 (2026-06-08)  ·
W-14 (2026-06-15)  ·
W-13 (2026-06-22)  ·
W-12 (2026-06-29)  ·
W-11 (2026-07-06)  ·
W-10 (2026-07-13)  ·
W-09 (2026-07-20)  ·
W-08 (2026-07-27)  ·
W-07 (2026-08-03)  ·
W-06 (2026-08-10)  ██ C  Běh okolo Brd (Trail · Příbram, race Sun 08-16)
W-05 (2026-08-17)  ·
W-04 (2026-08-24)  ·
W-03 (2026-08-31)  ·
W-02 (2026-09-07)  ░░
W-01 (2026-09-14)  ░░
W-00 (2026-09-21)  ██ A  Berlin Marathon (race Sun 09-27) ★
```

## Notes

- Yes, Sunday races display on the previous week's Monday-anchored line. That's fine; the line label is the ISO week's Monday. Race summary makes the actual date clear.
- If there's a race in W-00 that isn't the goal, list it separately with `★` reserved for the goal.
````

- [ ] **Step 2: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references/markdown-view-b.md`

Expected: file exists.

---

### Task 17: Create `references/html-template.html`

**Files:**
- Create: `SKILL_ROOT/race-render/references/html-template.html`

- [ ] **Step 1: Write the file**

Content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{GOAL_RACE_NAME}} — Race Plan</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      max-width: 1100px;
      margin: 2rem auto;
      padding: 0 1.5rem;
      color: #222;
      background: #fafafa;
    }
    header {
      border-bottom: 2px solid #333;
      padding-bottom: 1rem;
      margin-bottom: 2rem;
    }
    header h1 { margin: 0; font-size: 2rem; }
    header .meta {
      color: #666;
      margin-top: 0.25rem;
      font-size: 1rem;
    }
    header .countdown {
      font-size: 1.25rem;
      font-weight: 600;
      color: #b53a3a;
      margin-top: 0.5rem;
    }
    section { margin-bottom: 2.5rem; }
    section h2 {
      font-size: 1.1rem;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      color: #666;
      border-bottom: 1px solid #ddd;
      padding-bottom: 0.25rem;
    }
    .timeline-strip {
      position: relative;
      height: 60px;
      background: #eee;
      border-radius: 4px;
      margin: 1rem 0;
    }
    .timeline-strip .taper {
      position: absolute;
      top: 0; bottom: 0;
      background: rgba(255, 200, 200, 0.5);
    }
    .timeline-strip .marker {
      position: absolute;
      top: 8px;
      height: 44px;
      width: 8px;
      border-radius: 4px;
      cursor: pointer;
    }
    .timeline-strip .marker.A { background: #d64545; }
    .timeline-strip .marker.B { background: #e08a3c; }
    .timeline-strip .marker.C { background: #888; }
    .timeline-strip .marker:hover::after {
      content: attr(data-tooltip);
      position: absolute;
      bottom: 100%;
      left: 50%;
      transform: translateX(-50%);
      background: #222;
      color: #fff;
      padding: 4px 8px;
      border-radius: 4px;
      white-space: nowrap;
      font-size: 0.85rem;
      margin-bottom: 4px;
    }
    .month-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
      gap: 1rem;
    }
    .month {
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 6px;
      padding: 0.75rem;
    }
    .month h3 {
      margin: 0 0 0.5rem;
      font-size: 1rem;
      text-align: center;
    }
    .month table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
    .month th, .month td { text-align: center; padding: 0.2rem; }
    .month th { color: #888; font-weight: 500; }
    .month td.race { border-radius: 3px; font-weight: 600; color: #fff; }
    .month td.race.A { background: #d64545; }
    .month td.race.B { background: #e08a3c; }
    .month td.race.C { background: #888; }
    .month td.taper { background: rgba(255, 200, 200, 0.4); }
    table.race-list {
      width: 100%;
      border-collapse: collapse;
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 6px;
      overflow: hidden;
    }
    table.race-list th, table.race-list td {
      padding: 0.6rem 0.8rem;
      text-align: left;
      border-bottom: 1px solid #eee;
    }
    table.race-list th { background: #f0f0f0; font-size: 0.85rem; text-transform: uppercase; letter-spacing: 0.05em; color: #666; }
    table.race-list tr:last-child td { border-bottom: none; }
    .warnings {
      background: #fff5e5;
      border-left: 4px solid #e08a3c;
      padding: 0.75rem 1rem;
      border-radius: 4px;
      margin-bottom: 2rem;
    }
    .warnings h2 { margin-top: 0; color: #b06d20; border: none; text-transform: none; letter-spacing: normal; font-size: 1rem; }
    .warnings ul { margin: 0; padding-left: 1.2rem; }
    footer {
      color: #999;
      font-size: 0.85rem;
      margin-top: 3rem;
      text-align: center;
    }
  </style>
</head>
<body>
  <header>
    <h1>{{GOAL_RACE_NAME}}</h1>
    <div class="meta">{{GOAL_RACE_DATE_HUMAN}} · {{GOAL_RACE_LOCATION}} · {{GOAL_RACE_TYPE}}</div>
    <div class="countdown">{{DAYS_TO_GOAL}} days to go</div>
  </header>

  <!-- WARNINGS_BLOCK -->

  <section>
    <h2>Timeline</h2>
    <div class="timeline-strip">
      <!-- TAPER_OVERLAY -->
      <!-- RACE_MARKERS -->
    </div>
  </section>

  <section>
    <h2>Months</h2>
    <div class="month-grid">
      <!-- MONTH_BLOCKS -->
    </div>
  </section>

  <section>
    <h2>All races</h2>
    <table class="race-list">
      <thead>
        <tr><th>Date</th><th>Priority</th><th>Name</th><th>Type</th><th>Location</th><th>Role</th></tr>
      </thead>
      <tbody>
        <!-- RACE_ROWS -->
      </tbody>
    </table>
  </section>

  <footer>Generated by race-render on {{GENERATED_AT}}</footer>
</body>
</html>
```

- [ ] **Step 2: Verify no template tokens are broken**

Run:

```bash
grep -c "{{" /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references/html-template.html
grep -c "<!--" /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references/html-template.html
```

Expected: `{{` tokens ≥ 5 (five placeholders), HTML comments ≥ 4 (four render slots).

## Template tokens to fill in (`race-render` must replace these)

- `{{GOAL_RACE_NAME}}` — plain text.
- `{{GOAL_RACE_DATE_HUMAN}}` — e.g., `Sunday, September 27, 2026`.
- `{{GOAL_RACE_LOCATION}}` — `City, CC`.
- `{{GOAL_RACE_TYPE}}` — friendly label (see markdown-view-a.md).
- `{{DAYS_TO_GOAL}}` — integer, days between today and goal.
- `{{GENERATED_AT}}` — `YYYY-MM-DD HH:MM` local time.

## Comment slots to inject content into

- `<!-- WARNINGS_BLOCK -->` — replace with `<div class="warnings"><h2>⚠ Warnings</h2><ul><li>...</li></ul></div>` if any warnings; otherwise leave empty (remove the comment line entirely).
- `<!-- TAPER_OVERLAY -->` — replace with one or more `<div class="taper" style="left: X%; width: Y%"></div>` — `X%` is (taper_start - plan_start) / plan_span, `Y%` is taper_weeks / plan_span_weeks.
- `<!-- RACE_MARKERS -->` — one `<div class="marker <priority>" style="left: X%" data-tooltip="Name — Date"></div>` per race.
- `<!-- MONTH_BLOCKS -->` — one `<div class="month">...</div>` per month from plan start to goal race, with a mini-calendar table inside.
- `<!-- RACE_ROWS -->` — one `<tr>` per race.

- [ ] **Step 3: Verify**

Run: `ls /Users/stefan.pacinda@rossum.ai/.claude/skills/race-render/references/`

Expected: shows `markdown-view-a.md`, `markdown-view-b.md`, `html-template.html`.

---

### Task 18: Phase 3 verification scenario

- [ ] **Step 1: In a fresh Claude Code session (cwd = `REPO_ROOT`), invoke:**

```
Render my [race name] plan as all three views.
```

Expected: Claude invokes `race-render`, finds the plan, produces markdown-detailed, markdown-weekly, and writes HTML.

- [ ] **Step 2: Verify markdown-detailed output**

Check the conversation output:
- One block per race, ordered by date.
- Weeks-between annotations between race blocks.
- Taper annotation before the goal race.
- Goal race marked with `★`.
- If warnings exist, `⚠ Warnings` block is present.

- [ ] **Step 3: Verify markdown-weekly output**

Check:
- Header line.
- One line per week from plan start Monday to goal-race week.
- Race weeks show `██` and race summary.
- Taper weeks show `░░`.
- Idle weeks show `·`.
- Goal week has `★`.

- [ ] **Step 4: Verify HTML output**

Run:

```bash
ls /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders/
grep -c "{{" /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders/*.html
grep -c "<!--" /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders/*.html
```

Expected:
- HTML file exists.
- Zero `{{` tokens (all placeholders substituted).
- Zero uncommented `<!--` render-slot comments (all filled in or removed).

Open the HTML in a browser (`open renders/<file>.html` on macOS) and confirm:
- Header shows correct name, date, countdown.
- Timeline strip has markers colored A/B/C.
- Taper window shaded.
- Month grid shows race days highlighted.
- Race table lists all races.

- [ ] **Step 5: Commit the render**

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
git add renders/
git commit -m "feat: initial HTML render of race plan"
```

---

## Phase 4: End-to-end verification

### Task 19: Fresh-session full workflow test

**Purpose:** simulate a real user starting from scratch.

- [ ] **Step 1: Set up a clean test scenario**

Move current data aside so we test from scratch:

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
mkdir -p .test-backup
mv data/races/*.json .test-backup/ 2>/dev/null
mv plans/*.md .test-backup/ 2>/dev/null
mv renders/*.html .test-backup/ 2>/dev/null
```

- [ ] **Step 2: In a fresh Claude Code session, invoke a natural full-workflow prompt**

```
I want to plan my fall race season. Help me put together a calendar.
```

Expected sequence:
1. Claude recognizes race-planning intent, invokes `race-plan`.
2. `race-plan` checks `data/races/`, finds it empty, offers to invoke `race-scrape`.
3. `race-scrape` runs, populates `data/races/`.
4. Back in `race-plan`, walks Phase 2 questions.
5. Proposes build-ups, user picks.
6. Writes plan.
7. Offers to render — user picks all views.
8. `race-render` produces the three outputs.

- [ ] **Step 3: Confirm all artifacts exist**

Run:

```bash
ls /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/data/races/*.json
ls /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/plans/*.md
ls /Users/stefan.pacinda@rossum.ai/dev/repository/github/running/renders/*.html
```

Expected: at least one file in each directory.

- [ ] **Step 4: Confirm no taper violations**

Manually inspect the plan file. Confirm no build-ups fall inside the taper window relative to the goal race (unless the plan contains an explicit `warning:` field on that build-up).

- [ ] **Step 5: Test the "iteration" pathway**

In a new session:

```
Let me change something on my plan — swap out one of the build-ups.
```

Expected: Claude loads the existing plan, asks which build-up to swap, offers alternatives, updates the plan, backs up the previous version to `plans/archive/`.

Verify: `plans/archive/<name>.<timestamp>.md.bak` exists.

- [ ] **Step 6: Restore any pre-test data (optional)**

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
# Only if you want to keep prior test artifacts around:
# mv .test-backup/* data/races/ plans/ renders/ (be careful with paths)
rm -rf .test-backup
```

- [ ] **Step 7: Final commit**

```bash
cd /Users/stefan.pacinda@rossum.ai/dev/repository/github/running
git add -A
git commit -m "feat: end-to-end race calendar workflow verified"
```

---

## Self-review notes (from plan-writing phase)

- All spec sections mapped to tasks: skill decomposition → Phase 0 + skill directories; race-scrape → Tasks 2–7; race-plan → Tasks 8–13; race-render → Tasks 14–18; extensibility hooks → embedded in schema files (`plan-schema.md`'s `training_phase_by_week`, `foreign-sources.md`).
- No placeholders: every file's content is inlined in the plan.
- Type consistency: race `type` enum matches spec (`road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`) across scrape output, plan schema, and render friendly-label mapping.
- Freshness threshold consistent (7 days) in scrape SKILL.md and repo README.
- Priority enum consistent (A/B/C) across plan schema, taper rules, build-up heuristics, render markers.
- File paths use absolute paths for skill files (outside repo) and repo-relative paths for data/plans/renders.

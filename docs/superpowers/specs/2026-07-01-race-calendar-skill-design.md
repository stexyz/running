# Race Calendar Skill — Design

**Date:** 2026-07-01
**Status:** Approved for planning
**Author:** Štefan Pacinda (with Claude)

## Summary

A set of three composable, markdown-based Claude skills that help plan a race calendar iteratively: scrape Czech race listings, plan a season around a goal race with sensible build-ups and mandatory tapering, and render the resulting plan as a markdown timeline (for iteration) or a self-contained HTML calendar view (for the polished output). Data and plans persist in this repo so sessions build on each other.

## Goals

- Iteratively plan a race season around one or more goal races.
- Automatically pull race listings from major Czech sources so we don't hand-curate.
- Enforce tapering rules — especially before A-races — so the plan is training-sound.
- Produce a usable visual output: markdown for editing, HTML for reference.
- Persist state so we can revisit and iterate across sessions.
- Leave clean hooks for future integrations (foreign races, TrainingPeaks, Strava).

## Non-goals (v1)

Explicitly deferred. Documented here so v2 knows where to pick up:

- Scraping foreign race sites (DE / AT / SK / PL). Research pointer only in v1 — see `references/foreign-sources.md`.
- TrainingPeaks / Strava integration for current fitness / weekly mileage / training-phase context. Schema hook reserved but nothing wired up.
- Auto-refresh / cron / background jobs / notifications.
- Multi-user or shared plans.
- Any UI beyond the generated HTML file.
- Race registration, payment, or logistics tracking.

## Architecture

Three skills, each with one purpose, communicating through files in the `running` repo. No shared code, no runtime state — everything is on disk.

```
~/.claude/skills/
├── race-scrape/     — fetches races from 3 Czech sites, normalizes, writes data/races/*.json
├── race-plan/       — interactive planning: goal race + build-ups + tapering rules; writes plans/*.md
└── race-render/     — takes a plan file, produces markdown views or self-contained HTML

/running/  (this repo)
├── data/races/
│   ├── ceskybeh-2026.json
│   ├── top4running-2026.json
│   ├── svetbehu-2026.json
│   └── README.md              # schema, refresh notes, future sources list
├── plans/
│   ├── 2026-berlin-marathon.md
│   └── archive/               # completed plans
└── renders/
    └── 2026-berlin-marathon.html
```

**Composition in a session:**

- *"Plan my spring marathon"* → invokes `race-plan` → reads `data/races/` (invokes `race-scrape` if stale/missing) → writes/updates a plan file → offers to invoke `race-render`.
- *"Refresh race data"* → invokes `race-scrape` standalone.
- *"Render my Berlin plan as HTML"* → invokes `race-render` standalone.

## Shared race data schema

Written by `race-scrape`, consumed by `race-plan`. One JSON file per source.

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

**Race `type` enum:** `road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`.

**Location:** `region` may be `null` if not extractable. `country` defaults to `CZ` for v1 sources.

## Skill 1: `race-scrape`

**Purpose:** fetch race listings from three Czech sites, normalize them into the shared schema, and write JSON files to `data/races/`. No filtering, no planning logic.

**Trigger phrases:** "refresh race data", "scrape races", "update race calendar data". Also invoked by `race-plan` when data is stale/missing.

**Sources (v1):**
- https://top4running.cz/pg/kalendar-bezeckych-zavodu
- https://ceskybeh.cz/terminovka/
- https://www.svetbehu.cz/terminovka/

**Freshness threshold:** 7 days. Files with `fetched_at` older than 7 days trigger a refresh.

**Execution steps (defined in `SKILL.md`):**

1. For each source, check `fetched_at` in the corresponding JSON file. If missing or older than 7 days, mark for refresh.
2. For each source needing refresh, use `WebFetch` on the calendar URL. Extract name, date, location, distance, URL, and inferable type.
3. Normalize:
   - Parse Czech dates (e.g., "15. dubna 2026" → "2026-04-15").
   - Classify `type` heuristically from name and distance (see `references/type-classification.md`).
   - Extract city and (where possible) region; leave region `null` if unclear.
4. Write one JSON file per source with fetched races + `fetched_at` timestamp.
5. Report: `"Refreshed 3 sources, 847 races total, X new since last fetch."`

**Failure handling:**
- If a source fails (network, layout change), keep the previous JSON in place, log the error, warn the user.
- `race-plan` continues with the sources that succeeded.

**Skill directory:**

```
~/.claude/skills/race-scrape/
├── SKILL.md
└── references/
    ├── source-notes.md        # per-site quirks: date formats, expected structure
    ├── type-classification.md # heuristics for road/trail/ultra/etc classification
    ├── foreign-sources.md     # v2 candidates: laufkalender.de, runaustria.at, beh.sk, maratonypolskie.pl
    └── sample-outputs/        # anonymized snippets of expected scraped structure
```

## Skill 2: `race-plan`

**Purpose:** the core interactive skill — build and iterate on a race calendar around a goal race.

**Trigger phrases:** "plan my race calendar", "plan build-up races for [race]", "iterate on my [race] plan".

### Session flow

**Phase 1 — Identify context:**
List `plans/*.md`. Ask: "Iterate on an existing plan or start a new one?" If existing → load frontmatter, skip to Phase 4 in "modify" mode.

**Phase 2 — Gather goal race (new plans only):**

Ask one question at a time:

- **Goal race:** name, date, location. Search `data/races/` and offer matches; allow manual entry for foreign races.
- **Race type & distance:** pre-fill if the goal is in the data.
- **Priority level:** A (peak for this) / B (important but not peak) / C (train through).

**Phase 3 — Gather constraints:**

- **Regions willing to travel to.** Default: CZ-wide, plus DE / AT / SK / PL where public transport or highway connection is good. Free text; can pre-fill from prior plans.
- **Blackout dates.** Specific dates or weekends unavailable (family, work, travel).
- **Number of build-up races.** Rough count, or "let Claude suggest". Default: 3–5 for a marathon block.
- **Current fitness sanity check.** Free text (e.g., "coming back from injury", "off a good spring block"). No integration in v1; informs Claude's suggestions.

**Phase 4 — Propose build-ups:**

Read `data/races/`, filter by region, date window, and appropriate type. Apply build-up heuristics from `references/build-up-heuristics.md`:

- **Road marathon (A) goal:** HM ~4–6 weeks out, 10k ~8–10 weeks out, optional tune-up 2–3 weeks out (short, easy).
- **Trail / ultra (A) goal:** shorter trail race 4–6 weeks out; optional vertical or technical race earlier for skill exposure.
- **Short road (5k/10k) (A) goal:** 5k time trial or parkrun ~2 weeks out; another 5k/10k ~4–5 weeks out for pacing.

**Taper rules — non-negotiable pre-A-race:**

| Goal distance | Taper window (no hard races) |
|---|---|
| Marathon / ultra | 3 weeks |
| Half-marathon | 2 weeks |
| 10k | ~10 days |
| 5k | 1 week |

- **Pre-B-race taper:** soft rule — 1 week for HM / marathon B-races, a few days for shorter. Skill warns but allows.
- **C-races:** no taper, meant to be trained through. Freely placed.
- **Minimum spacing between hard races:** ~2 weeks. Skill warns on closer spacing.

Present 2–3 candidate build-up calendars with rationale. Example: *"Option 1: HM in Karlovy Vary 5 weeks out — pacing rehearsal, easy train from Prague."*

**Phase 5 — Write the plan:**
Save to `plans/<year>-<goal-race-slug>.md` using the schema in the next section.

**Phase 6 — Offer to render:**
"Plan written. Want me to render markdown views and/or HTML?" → invokes `race-render`.

### Iteration mode

On re-invocation with an existing plan: load it, jump to a "what would you like to change?" prompt. Support: swap a build-up, add/remove a race, adjust priorities, extend the plan window.

### Plan file schema

Frontmatter + human-editable body.

```markdown
---
goal_race:
  name: Berlin Marathon
  date: 2026-09-27
  location: Berlin, DE
  type: road-marathon
  priority: A
build_ups:
  - name: Pražský půlmaraton
    date: 2026-04-05
    location: Praha, CZ
    type: road-half
    priority: B
    role: pacing rehearsal
  - name: Běh okolo Brd
    date: 2026-08-16
    location: Příbram, CZ
    type: trail
    priority: C
    role: long training run with support
constraints:
  regions: [CZ, DE, AT, SK, PL]
  blackouts: [2026-06-14, 2026-07-19]
  notes: "Coming off spring block, no injuries"
# Reserved for future TrainingPeaks / Strava integration — unused in v1
training_phase_by_week: {}
---

# Plan notes

Free-form notes: training thoughts, why each race was picked, etc.
```

### Skill directory

```
~/.claude/skills/race-plan/
├── SKILL.md
└── references/
    ├── build-up-heuristics.md   # per-goal-type build-up patterns and spacing rules
    ├── taper-rules.md           # the taper table above, elaborated
    ├── plan-schema.md           # required and optional plan file fields
    └── czech-regions.md         # region names, rough travel-time hints from Prague / Brno
```

## Skill 3: `race-render`

**Purpose:** turn a plan file into a visual representation. Two modes: markdown (for iteration) and HTML (polished final view).

**Trigger phrases:** "render my [race] plan", "show my calendar as HTML", "generate calendar view".

### Markdown render — two views

**View A — Detailed vertical timeline (default):**

Linear, one block per race, with weeks-between annotations and taper-window callouts:

```
2026 — Berlin Marathon Block

  Apr 05 (Sun)  ┃ B ┃ Pražský půlmaraton      HM · Praha, CZ
                │       role: pacing rehearsal
                │
                │  ↕ 19 weeks
                │
  Aug 16 (Sun)  ┃ C ┃ Běh okolo Brd            Trail · Příbram, CZ
                │       role: long training run
                │
                │  ↕ 6 weeks   ← 3-week A-race taper starts Sep 06
                │
  Sep 27 (Sun)  ┃ A ┃ Berlin Marathon          Marathon · Berlin, DE  ★
```

Preceded by a compact table (date · priority · name · type · location).

**View B — Proportional week grid:**

One line per calendar week from plan start to goal race, including idle weeks, so visual distance reflects real time. Races appear on their week's line. Taper window shaded. Example row style:

```
W-24 (2026-04-05)  ██ B  Pražský půlmaraton (HM · Praha)
W-23 (2026-04-12)  ·
W-22 (2026-04-19)  ·
...
W-06 (2026-08-16)  ██ C  Běh okolo Brd (Trail · Příbram)
W-05 (2026-08-23)  ·
W-04 (2026-08-30)  ·
W-03 (2026-09-06)  ░░  ← taper
W-02 (2026-09-13)  ░░  ← taper
W-01 (2026-09-20)  ░░  ← taper
W-00 (2026-09-27)  ██ A  Berlin Marathon ★
```

Both views end with a **warnings** section that lists any rule violations:

```
⚠ Warnings
  • Race on 2026-09-13 falls inside the 3-week A-race taper window
  • Only 8 days between build-up on 2026-08-16 and follow-up on 2026-08-24
```

### HTML render — polished final view

A single self-contained HTML file — no external dependencies, all styles inline. Standard layout (can be iterated later):

- **Header:** goal race name, date, days-to-race countdown.
- **Timeline strip:** horizontal bar spanning the plan window, race markers colored by priority (A red, B orange, C gray), taper window shaded.
- **Month grid:** actual months from plan start through goal-race month, each a mini calendar with race days highlighted.
- **Race list:** sortable table at bottom.

Vanilla HTML/CSS/JS, no build step. Written to `renders/<plan-slug>.html`.

### Skill directory

```
~/.claude/skills/race-render/
├── SKILL.md
└── references/
    ├── markdown-view-a.md       # detailed timeline format
    ├── markdown-view-b.md       # proportional week grid format
    └── html-template.html       # base HTML/CSS with placeholders
```

## Error handling and validation

- **Stale or missing race data:** `race-plan` warns and offers to invoke `race-scrape`.
- **Failed scrape (single source):** keep last-good JSON, log the error, continue.
- **Malformed plan frontmatter:** `race-render` validates required fields and reports the offending field by name.
- **Ambiguous race match in Phase 2:** present numbered candidates, ask for a pick.
- **Taper-rule violations:** never silent — always surfaced in the warnings section of the render and in the interactive planning session.

## Verification

Each skill's `SKILL.md` ends with a short "How to verify this worked" section listing concrete checks (e.g., "confirm `data/races/ceskybeh-2026.json` exists and `fetched_at` is recent; confirm at least 100 races").

No formal test suite — skills are markdown, verification is by running them.

## Extensibility hooks (for v2)

- **Foreign scraping:** `references/foreign-sources.md` in `race-scrape` lists laufkalender.de (DE), runaustria.at (AT), beh.sk (SK), maratonypolskie.pl (PL). Adding a source means: add source ID + normalize function guidance + entry in the shared schema. No structural changes needed.
- **TrainingPeaks / Strava:** plan frontmatter reserves `training_phase_by_week: {}`. Future integration populates it (base / build / peak / taper / recovery per week). Rendering can then show phase context alongside races.
- **Data schema is stable:** any new source must produce the shared race schema. New race types added to the enum are additive, not breaking.

## Open questions

None at spec time. All Section-level questions during brainstorming resolved.

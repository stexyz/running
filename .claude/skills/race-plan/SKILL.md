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

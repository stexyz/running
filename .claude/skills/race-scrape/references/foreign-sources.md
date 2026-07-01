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

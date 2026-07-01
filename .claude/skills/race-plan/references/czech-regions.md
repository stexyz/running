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

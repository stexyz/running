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

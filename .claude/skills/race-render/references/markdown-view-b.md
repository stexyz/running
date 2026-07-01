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

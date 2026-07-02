---
goal_race:
  name: Dresden Marathon
  date: 2026-10-25
  location: Dresden, DE
  type: road-marathon
  priority: A
build_ups:
  - name: Běhej lesy #4 Klínovec - ČSOB 23 km
    date: 2026-07-11
    location: Loučná pod Klínovcem, CZ
    type: trail
    priority: C
    role: Long trail day, terrain exposure. Verify elevation ≤2000m before signup.
    source: behej-2026-07-11-behej-lesy-4-klinovec-csob-23-km
  - name: Mizuno Neon Run Praha
    date: 2026-08-29
    location: Praha, CZ
    type: road-5k
    priority: C
    role: Late-summer sharpener after August dead zone.
    source: behej-2026-08-29-mizuno-neon-run-praha
  - name: Birell Večerní Běh na 10 km Praha
    date: 2026-09-05
    location: Praha 1, CZ
    type: night-race
    priority: B
    role: 10k threshold, calibrate current fitness ~7 weeks out.
    source: ceskybeh-2026-09-05-birell-vecerni-beh-na-10-km-praha
  - name: ŠKODA FIT půlmaraton 2026
    date: 2026-09-20
    location: Plzeň, CZ
    type: road-half
    priority: B
    role: Main pacing rehearsal, marathon-pace test 5 weeks out.
    source: sportbase-2026-09-20-skoda-fit-pulmaraton-2026
constraints:
  regions: [CZ, DE]
  blackouts:
    - 2026-07-18
    - 2026-07-19
    - 2026-08-08
    - 2026-08-09
    - 2026-08-15
    - 2026-08-16
    - 2026-08-22
    - 2026-08-23
  notes: |
    Prefer CZ races within 2h of Prague. Goal race Dresden is DE but within 2h.
    Aug 1-2 available as a soft blackout: OK for C-race intensity only due to jet lag; nothing scheduled there.
    No fitness or injury notes provided.
training_phase_by_week: {}
---

# Plan notes

Iteration 1 — initial plan.

## Structure

- **Jul 11 Klínovec 23km trail (C):** long trail day, terrain rehearsal, early-season fitness marker.
- **Aug 1–23:** family/travel commitments, no racing.
- **Aug 29 Neon Run 5k (C):** sharpener to shake off the dead zone.
- **Sep 5 Birell 10k evening (B):** first hard race back, threshold calibration. RunCzech event, flat central Prague course.
- **Sep 20 Plzeň HM (B):** main marathon-pace rehearsal. 5 weeks out.
- **Oct 4–24:** 3-week A-race taper. No hard races.
- **Oct 25 Dresden Marathon (A):** ★

## Data-quality issues noted from the scrape

- Dresden Marathon record has `location.city = "Dresden, Německo"` and `location.country = "CZ"` — should be city "Dresden", country "DE". Follow-up: improve `race-scrape/references/source-notes.md` to handle city-with-country strings and correct the country field for foreign races.
- Bechovice 10k appears twice in `all-2026.json` (dedup miss between "Běchovice - Praha" and "Česká spořitelna Běchovice-Praha, MČR"). Not used here but worth flagging.
- Neon Run Praha is classified as both `road-5k` (behej source) and `trail` (ceskybeh source) — inconsistent, should be road-5k.
- ŠKODA FIT Plzeň appears at multiple distances across multiple source scrapers (1k, 2k, 5k, 10k, 21.1k). The 21.1km entry from `sportbase` was used.

## Alternatives considered

- **Bechovice 10k (Sep 27):** dropped in favor of Birell 10k (Sep 5). Reason: Sep 27 sits inside the 4-week pre-goal window and the only nearby HM (Oct 4 Hradec) would violate the 3-week A-race taper.
- **Jihlavský půlmaraton (Sep 6):** dropped — would clash back-to-back with Birell Sep 5.

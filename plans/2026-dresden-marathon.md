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
  - name: Half Marathon Praha 13
    date: 2026-08-28
    location: Praha 13, CZ
    type: road-half
    priority: B
    role: Early-season HM pacing check, 8 weeks out. Replaces Aug 29 Neon Run 5k.
    source: behej-2026-08-28-half-marathon-praha-13
    warning: 8 days before Sep 5 Birell 10k (B) — violates 14-day A/B spacing rule. Accepted by user.
  - name: Birell Večerní Běh na 10 km Praha
    date: 2026-09-05
    location: Praha 1, CZ
    type: night-race
    priority: B
    role: 10k threshold, calibrate current fitness ~7 weeks out.
    source: ceskybeh-2026-09-05-birell-vecerni-beh-na-10-km-praha
    warning: 8 days after Aug 28 Praha 13 HM (B) — violates 14-day A/B spacing rule. Accepted by user.
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
    - 2026-08-29
  notes: |
    Prefer CZ races within 2h of Prague. Goal race Dresden is DE but within 2h.
    Aug 1-2 available as a soft blackout: OK for C-race intensity only due to jet lag; nothing scheduled there.
    No fitness or injury notes provided.
training_phase_by_week: {}
---

# Plan notes

Iteration 2 — added a second HM at user's request.

## Structure

- **Jul 11 Klínovec 23km trail (C):** long trail day, terrain rehearsal, early-season fitness marker.
- **Aug 1–23:** family/travel commitments, no racing.
- **Aug 28 Praha 13 HM (B):** early-season HM pacing check, 8 weeks out. Friday evening event.
- **Sep 5 Birell 10k evening (B):** RunCzech event, flat central Prague course. Threshold calibration.
- **Sep 20 Plzeň HM (B):** main marathon-pace rehearsal, 5 weeks out.
- **Oct 4–24:** 3-week A-race taper. No hard races.
- **Oct 25 Dresden Marathon (A):** ★

## Warnings

- **Aug 28 → Sep 5 spacing = 8 days.** Violates the 14-day A/B hard-spacing rule. Both entries carry `warning:` fields. Coach's mitigation options for next iteration: (a) downgrade Birell 10k to a C-race and run it as controlled tempo, (b) drop Birell entirely, or (c) accept as a two-race "block" and back off training volume in the intervening week.
- The Aug 29 Mizuno Neon Run 5k was removed to make room for the Aug 28 HM (adjacent days).

## Data-quality issues noted from the scrape

- Dresden Marathon record has `location.city = "Dresden, Německo"` and `location.country = "CZ"` — should be city "Dresden", country "DE". Follow-up: improve `race-scrape/references/source-notes.md` to handle city-with-country strings and correct the country field for foreign races.
- Bechovice 10k appears twice in `all-2026.json` (dedup miss between "Běchovice - Praha" and "Česká spořitelna Běchovice-Praha, MČR"). Not used here but worth flagging.
- Neon Run Praha is classified as both `road-5k` (behej source) and `trail` (ceskybeh source) — inconsistent, should be road-5k.
- ŠKODA FIT Plzeň appears at multiple distances across multiple source scrapers (1k, 2k, 5k, 10k, 21.1k). The 21.1km entry from `sportbase` was used.

## Alternatives considered

- **Bechovice 10k (Sep 27):** dropped in favor of Birell 10k (Sep 5). Reason: Sep 27 sits inside the 4-week pre-goal window and the only nearby HM (Oct 4 Hradec) would violate the 3-week A-race taper.
- **Jihlavský půlmaraton (Sep 6):** dropped — would clash back-to-back with Birell Sep 5.
- **Sep 12 HM (Bezdězský / Okolo Kolovrat) as C long tempo:** rejected in favor of Aug 28 Praha 13. Would have violated 14-day spacing on *both* sides (Birell Sep 5 → Plzeň Sep 20).
- **Aug 22 HM (Stadlern DE / Hrabovský Ostrava):** rejected — falls on user's Aug 22–23 blackout.

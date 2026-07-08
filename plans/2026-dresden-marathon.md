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
  - name: Borovská desítka
    date: 2026-07-25
    location: Havlíčkova Borová, CZ
    type: road-10k
    priority: C
    role: Aerobic sharpener 14 weeks out, 2 weeks after Klínovec. Bridges the Jul→Sep gap; run at controlled effort (marathon-pace to threshold), not race pace.
    source: svetbehu-2026-07-25-borovska-desitka
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
    - 2026-08-28
    - 2026-08-29
  notes: |
    Prefer CZ races within 2h of Prague. Goal race Dresden is DE but within 2h.
    Aug 1-2 available as a soft blackout: OK for C-race intensity only due to jet lag; nothing scheduled there.
    No fitness or injury notes provided.
training_phase_by_week: {}
---

# Plan notes

Iteration 5 — taper rule shortened from 3 weeks to 2 weeks for
road-marathon (updated in `.claude/skills/race-plan/references/`).
Taper window is now Oct 11 – Oct 24 (was Oct 4 – Oct 24). This opens
Oct 4 as a viable hard-race slot 3 weeks out from Dresden.

Iteration 4 — full replan against updated blackouts. Added a Jul 25
C-race sharpener (Borovská desítka 10k) to bridge the 8-week Klínovec →
Birell gap. Aug is a full dead zone: all four weekends are personal
blackouts (Aug 1–2 soft, 8–9, 15–16, 22–23, 28–29).

## Structure

- **Jul 11 Klínovec 23km trail (C):** long trail day, terrain rehearsal, early-season fitness marker.
- **Jul 25 Borovská desítka 10k (C):** aerobic sharpener 14 wks out, controlled effort. 2 weeks after Klínovec.
- **Aug:** dead zone (family/travel commitments across every weekend).
- **Sep 5 Birell 10k evening (B):** first hard race back, threshold calibration. RunCzech event, flat central Prague course.
- **Sep 20 Plzeň HM (B):** main marathon-pace rehearsal, 5 weeks out.
- **Oct 11–24:** 2-week A-race taper. No hard races.
- **Oct 25 Dresden Marathon (A):** ★

## Coaching notes

- Plan matches the standard marathon build-up heuristic (10k-B 8–10 wks
  out, HM-B 4–6 wks out, 3-wk taper).
- Aug 30 was considered as a second C-race (Memoriál Jana Fryce 10k,
  Neratovice) but rejected: only 6 days before Birell, would compromise
  Birell's 4–5 day pre-race taper.
- Borovská desítka has inconsistent source classification (road-10k vs
  trail across sources — Havlíčkova Borová is Vysočina, so mixed rolling
  terrain likely). Treated as road-10k here.

## Data-quality issues noted from the scrape

- Dresden Marathon record has `location.city = "Dresden, Německo"` and `location.country = "CZ"` — should be city "Dresden", country "DE". Follow-up: improve `race-scrape/references/source-notes.md` to handle city-with-country strings and correct the country field for foreign races.
- Bechovice 10k appears twice in `all-2026.json` (dedup miss between "Běchovice - Praha" and "Česká spořitelna Běchovice-Praha, MČR"). Not used here but worth flagging.
- Neon Run Praha is classified as both `road-5k` (behej source) and `trail` (ceskybeh source) — inconsistent, should be road-5k.
- ŠKODA FIT Plzeň appears at multiple distances across multiple source scrapers (1k, 2k, 5k, 10k, 21.1k). The 21.1km entry from `sportbase` was used.

## Alternatives considered

- **Bechovice 10k (Sep 27):** dropped in favor of Birell 10k (Sep 5). Reason: Sep 27 sits inside the 4-week pre-goal window; noise-vs-signal argument. Under the current 2-week taper rule, Sep 27 or Oct 4 (e.g., Hradec HM) are now valid hard-race slots to consider in a future iteration.
- **Jihlavský půlmaraton (Sep 6):** dropped — would clash back-to-back with Birell Sep 5.
- **Sep 12 HM (Bezdězský / Okolo Kolovrat) as C long tempo:** rejected in favor of Aug 28 Praha 13. Would have violated 14-day spacing on *both* sides (Birell Sep 5 → Plzeň Sep 20).
- **Aug 22 HM (Stadlern DE / Hrabovský Ostrava):** rejected — falls on user's Aug 22–23 blackout.

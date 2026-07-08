# Taper rules

Applied by `race-plan` when placing build-ups and when validating a plan.

## Pre-A-race taper — mandatory

No hard races (A or B) inside the taper window before an A-race. C-races are allowed if the user explicitly overrides.

| Goal race type          | Taper window (weeks) |
|-------------------------|---------------------|
| road-marathon           | 2                   |
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

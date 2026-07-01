# Coverage histogram render

Optional post-scrape visualization. Invoked from SKILL.md step 10 if the user opts in.

## Purpose

A single self-contained HTML file showing weekly counts of the three road-race distances (marathon / half-marathon / 10k) across the scraped window. Answers "how does the calendar concentrate?" at a glance — e.g. spotting a marathon-heavy weekend, or dead weeks for halves.

## Inputs

- `data/races/all-<year>.json` — the consolidated deduped file.
- `WINDOW_START`, `WINDOW_END` — from the current run.

## Output

`renders/race-coverage-<window-slug>.html`, where `<window-slug>` is `YYYYMMDD-YYYYMMDD` derived from the window (e.g. `20260701-20261031`).

After writing, `open` the file (macOS) so it appears in the user's default browser.

## Filter and bucketing

- Include only races with `type ∈ {"road-marathon", "road-half", "road-10k"}`.
- Include only races with `WINDOW_START ≤ date ≤ WINDOW_END`.
- Bucket by ISO week (Monday-start). Key = the Monday of that week (YYYY-MM-DD).

## Layout

- Weeks as columns, left-to-right chronological.
- Each column is a **stacked bar** of the three types. Bar height ∝ that week's total count of the three types. Max bar height = a fixed value (e.g. 320 px) mapped to the global max weekly total.
- Show total count above each bar.
- Include an X-axis tick showing the week's Monday as `"MMM DD"`.
- On hover of a column, show a tooltip listing every included race that week (`YYYY-MM-DD · Type · Name (City)`), one per line. No JS needed — CSS `:hover` on the column reveals a hidden `<div class="tooltip">`.

## Colors

Fixed palette, chosen for accessibility:
- `road-marathon` → `#c0392b` (red)
- `road-half` → `#2980b9` (blue)
- `road-10k` → `#27ae60` (green)

## Chrome

- Title: `Race coverage · road distances · <window as human string>`
- Meta line under title: source file path + a note that bar height = races that week.
- Legend row with color swatches and per-type totals for the window.
- Footer: `Rendered <fetched_at from JSON> · Total <N> races across <M> weeks · Distance classification per .claude/skills/race-scrape/references/type-classification.md`.

## Implementation notes

- Pure inline CSS. No external libraries, fonts, or images. The file must render offline.
- Use `-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif` as the font stack.
- Do not put more than ~30 weekly columns on one page without adjusting `min-width`; the layout uses `flex: 1` so it auto-fits.
- The tooltip uses `<pre>` with `white-space: pre` so one race per line.

## Verification

1. `renders/race-coverage-*.html` file exists and parses as valid HTML.
2. Sum of counts across bars equals the filtered race count from the JSON.
3. Hovering a random column shows a race list that matches manual inspection of that week in the JSON.

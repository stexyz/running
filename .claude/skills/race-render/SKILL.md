---
name: race-render
description: Use when the user wants to render, visualize, or view a race plan. Reads a plan file from plans/, produces a markdown timeline (two view options), or a self-contained HTML calendar page written to renders/.
---

# race-render

Takes a plan file from `plans/` and produces a visual representation. Two markdown views (for iteration) and one HTML view (for polished output).

**Repo assumption:** cwd has `plans/` and `renders/` directories.

## Trigger and inputs

When invoked, ask the user:

1. **Which plan?** List `plans/*.md` (skip `archive/`). If one exists, default to it. If multiple, ask.
2. **Which output?** Options:
   - `markdown-detailed` — vertical timeline with weeks-between annotations.
   - `markdown-weekly` — proportional week grid, one line per calendar week.
   - `html` — self-contained HTML page.
   - `all` — produce all three.

## Execution

1. Load and validate the plan file per `.claude/skills/race-plan/references/plan-schema.md`.
2. Compute derived data:
   - Sort all races (goal + build-ups) by date.
   - Compute gaps between consecutive races in weeks (rounded to 1 decimal if fractional).
   - Compute the pre-A-race taper window from `.claude/skills/race-plan/references/taper-rules.md`.
   - Identify rule violations (any race inside taper window, spacing under 2 weeks between hard races).
3. Render per the requested view(s) using the corresponding reference template.
4. Write outputs:
   - Markdown views → print to conversation directly (user is iterating).
   - HTML view → write to `renders/<plan-slug>.html`, then tell the user the path.
5. If violations exist, always emit a `⚠ Warnings` block, regardless of view.

## Loading references

- `references/markdown-view-a.md` — detailed timeline format and template.
- `references/markdown-view-b.md` — proportional week grid format and template.
- `references/html-template.html` — base HTML with placeholder tokens.

## How to verify this worked

1. For `markdown-detailed`: output contains one block per race, weeks-between annotations between them, taper-window annotation before the A-race, and (if applicable) a warnings block.
2. For `markdown-weekly`: output has one line per calendar week from plan start to goal race, with `██` markers on race weeks and `░░` on taper weeks.
3. For `html`: `renders/<plan-slug>.html` exists, opens in a browser, shows header + timeline strip + month grid + race table. No missing tokens (grep for `{{` or `%%` should return nothing).
4. Warnings block accurately lists all violations (or is absent if none).

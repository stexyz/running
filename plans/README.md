# Race plans

One file per goal race. Written and iterated on by the `race-plan` skill.

## Naming

`<year>-<goal-race-slug>.md` — e.g., `2026-berlin-marathon.md`. Lowercase, hyphenated.

Completed plans move to `archive/`.

## Structure

YAML frontmatter (machine-read by skills) + free-form markdown notes.

See the plan schema: `~/.claude/skills/race-plan/references/plan-schema.md`.

Minimal example:

```markdown
---
goal_race:
  name: Berlin Marathon
  date: 2026-09-27
  location: Berlin, DE
  type: road-marathon
  priority: A
build_ups: []
constraints:
  regions: [CZ, DE, AT, SK, PL]
  blackouts: []
  notes: ""
training_phase_by_week: {}
---

# Plan notes
```

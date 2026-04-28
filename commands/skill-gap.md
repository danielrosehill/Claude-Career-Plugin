---
description: Refresh skills/gaps.md by diffing current skill inventory against target-role profile. Carries plan notes forward.
---

# /career:skill-gap

Single-skill command. Runs the `skill-gap-analysis` skill.

## Usage

```
/career:skill-gap [--target=<file>] [--inventory=<file>]
```

## What it does

- Reads `skills/inventory.md` and `skills/target-profile.md`.
- Categorises every target skill as **critical** (gap ≥2), **material** (gap = 1), or **met**.
- Carries forward existing **plan** notes from the previous gaps.md where the skill still appears.
- Writes `skills/gaps.md` with `**Last refreshed:** YYYY-MM-DD`.

## Difference from /career:suggest skills

- `skill-gap` measures the inventory delta against a target profile.
- `/career:suggest skills` proposes a learning plan within a horizon.

## Notes

- Run after editing inventory or target profile.
- New gaps without plans are surfaced in the run summary.

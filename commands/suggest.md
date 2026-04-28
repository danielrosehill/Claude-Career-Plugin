---
description: Recommendation entry point — given ground-truth and discovery notes, suggest companies (with anti-loop), roles, skills to close, projects to ship, or consulting packages. All suggestions write a dated file to <WORKING_FOLDER>/recommendations/.
---

# /career:suggest

Five sub-skills, one entry point. Each writes a dated recommendations file so the user has a log of how thinking evolved.

## Subcommands

```
/career:suggest companies            (--domain=<slug> | --from=<note-slug> | --all-fresh) [--n=10] [--lens=employer|client|partner] [--include-recent] [--explain]
/career:suggest roles                [--company=<slug>] [--domain=<slug>] [--mode=existing|pitch|both] [--n=10]
/career:suggest skills               (--target-role=<title> | --target-company=<slug> | --target-domain=<slug>) [--horizon=<weeks>]
/career:suggest projects             [--close-gap=<skill>] [--target-company=<slug>] [--target-domain=<slug>] [--time-budget=weekend|week|month] [--n=5]
/career:suggest consulting-packages  [--domain=<slug>] [--target-company=<slug>] [--shape=audit|build|advisory|teach] [--n=5]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `companies` | `suggest-companies` |
| `roles` | `suggest-roles` |
| `skills` | `suggest-skills` |
| `projects` | `suggest-projects` |
| `consulting-packages` | `suggest-consulting-packages` |

## Anti-loop guarantee (companies only)

`suggest companies` reads `crm/outreach.md` and:
- **Hard-excludes** companies with outreach in the last 30 days (or any active thread within 90 days).
- **Rank-downs** stalled / cooling.
- **Surfaces** the freshness column on every row so the logic is visible.
- Two consecutive runs with no new signal will say "stable list" rather than silently repeating.

`--include-recent` disables the hard exclusion (rare; for review).

## End-to-end example

```
/career:discover by-domain "ai eval startups"
/career:suggest companies --domain=ai-eval-startups --explain
# top pick: Snowglobe (score 11, never-contacted)

/career:suggest roles --company=snowglobe --mode=both
/career:suggest skills --target-company=snowglobe --horizon=8
/career:suggest projects --close-gap="evals tooling" --time-budget=week
# now you have: target, role shape, skill plan, project to ship
```

## Notes

- All suggestions read `ground-truth.md`. If it's thin or missing required sections, run `/career:ground-truth edit` first.
- Output files are dated and never overwritten — `recommendations/` becomes a longitudinal log.
- `--explain` on `companies` shows per-row scoring rationale.

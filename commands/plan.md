---
description: Weekly planning + logging entry point. Reads workspace state to propose a week of actions, or reads diffs at end of week to produce a wrap-up. Delegates time-blocks/tasks to schedule-manager when installed.
---

# /career:plan

Two sub-skills, one command. The week-cadence loop: plan → execute → log → plan.

## Subcommands

```
/career:plan week [--week-of=YYYY-MM-DD] [--target=6] [--mix=balanced|outreach-heavy|build-heavy] [--dry-run]
/career:plan log  [--week-of=YYYY-MM-DD] [--quick]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `week` | `weekly-plan` |
| `log` | `weekly-log` |

## End-of-week / start-of-week loop

```
# Friday afternoon — close out
/career:plan log
# → plans/<date>-week-log.md (diff + reflection)

# Sunday evening — set up the week
/career:plan week --mix=balanced
# → plans/<date>-week-plan.md
# → if schedule-manager installed: tasks + events pushed (with confirmation)
```

## Output layout

```
plans/
  2026-04-21-week-plan.md
  2026-04-25-week-log.md
  2026-04-28-week-plan.md
  ...
```

## Prerequisites

- `ground-truth.md` populated.
- Some workspace state — outreach.md rows, opportunities.md entries, or recommendations files. A brand-new workspace will produce a "starting state" plan instead.
- Optional: `schedule-manager` companion plugin for calendar/task push. Falls back to markdown queue without it.

## Notes

- Every planned action cites a source row from CRM / opportunities / recommendations / inbox / meetings — no fabricated entries.
- Plan completion (X/Y actions done) is a fact, not a grade. Used as input to next week's plan.
- The "what I'm explicitly not doing this week" section in each plan forces honest scope.

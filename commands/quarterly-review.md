---
description: Quarterly career self-review — plan vs actual across goals, CPD, skills, applications.
---

# /career:quarterly-review

Produce a dated quarterly review comparing plan vs actual.

## Steps

1. Determine the quarter (`$ARGUMENTS` like `Q2-2026`, else current quarter).
2. Pull:
   - Goals from `goals/` active during the quarter.
   - CPD entries from `cpd-log/` for the quarter's months.
   - Skills diff: compare `skills/gaps.md` (current) against the version at quarter-start if stored; otherwise note absent baseline.
   - Applications: pipeline events from `data/processes.json` in that period (job-search workspaces).
3. Write `reviews/YYYY-Q<n>.md`:
   - What was planned.
   - What actually happened.
   - Goals to retire, adjust, add.
   - Skills meaningfully advanced.
   - CPD hours by category.
   - Application activity (if applicable).
   - Next-quarter intents.
4. Present a 5-bullet summary.

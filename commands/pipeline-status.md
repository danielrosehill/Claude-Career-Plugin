---
description: Summarise the current job-application pipeline — by stage, recency, and next actions.
---

# /career:pipeline-status

Read `data/processes.json` (job-search variant) and produce a status report.

## Steps

1. Load `data/processes.json`. If absent, tell the user to log a role first with `/career:log-role`.
2. Summarise:
   - **Total by status.**
   - **Recent activity (last 7 days).**
   - **Awaiting follow-up** (entries whose `next_date` is today or past).
   - **Upcoming interviews / deadlines** (next 14 days).
   - **Stale** (no update in 21+ days and status not terminal).
3. Present as a clear status report. Flag anything that needs action today.

# Outreach Log

**Append-only.** Every plugin recommendation reads this first. Do not rewrite history — use `/career:outreach status` to update an existing row's state.

| date | company | contact | channel | template | status | next-action | notes |
| --- | --- | --- | --- | --- | --- | --- | --- |

## Status values

- `sent` — message dispatched, no reply yet.
- `replied` — they responded.
- `meeting-booked` — call scheduled.
- `meeting-done` — call happened, awaiting follow-up.
- `stalled` — no movement for >30 days.
- `closed-won` — converted (job offer / signed contract).
- `closed-lost` — declined or dropped.

## Channels

- `email` (most common — via email-skills plugin).
- `linkedin`
- `intro` (via warm intro from another contact).
- `event` (met at conference / meetup / hackathon).
- `referral`
- `other` — explain in notes.

## Anti-loop guarantee

`/career:suggest companies` will:
- **Hard exclude** rows with status `sent` and `last-update < 90 days`.
- **Hard exclude** rows with `closed-won` / `closed-lost` / `stalled` and `last-update < 90 days`.
- **Rank-down** previously-stalled rows that are now older than 90 days.
- Always include never-before-contacted matches.

Output includes a `freshness` column so you can see why each row appears.

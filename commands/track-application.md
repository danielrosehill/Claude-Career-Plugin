---
description: Update the status of a job application in the pipeline — stage change, interview scheduled, rejection, offer, follow-up.
---

# /track-application

Maintain `data/processes.json` (job-search variant) or the equivalent roles tracking file.

## Steps

### 1. Identify the application

Parse `$ARGUMENTS` for company/slug. If absent, list pending applications from the pipeline DB and ask.

### 2. Capture the update

Ask (or parse inline):

- **New status:** `interested` / `applied` / `screening` / `interviewing` / `onsite` / `offer` / `negotiating` / `accepted` / `rejected` / `withdrawn` / `ghosted`.
- **Date of event:** default today.
- **Stage detail:** e.g. "recruiter screen scheduled for 2026-04-28" or "declined — comp gap".
- **Next action / next date:** the follow-up, if any.
- **Notes:** interviewer names, prep links, takeaways.

### 3. Update the pipeline DB

Read `data/processes.json`. Locate the matching entry (by slug or `company+title`). If not found, prompt the user to run `/log-role` first, or offer to create the entry inline.

Append the event to the entry's `events[]` array with `{date, stage, note, next_action, next_date}`. Update top-level `status` to the new status. Bump `updated_at`.

Write back with stable formatting (2-space indent, trailing newline).

### 4. Side-effects

- If status went to `interviewing` / `onsite`, remind the user they can run `/career:interview-prep` (from the job-search variant's future commands) or collect interview context manually.
- If status went to `offer`, suggest `/career:compare-offer` against any other tracked offers and `/career:salary-benchmark` on the role.
- If status went to `rejected`, ask if the user wants a short retrospective note logged to `outputs/retros/<date>-<slug>.md`.

### 5. Confirm

One-line confirmation with new status and the next scheduled action.

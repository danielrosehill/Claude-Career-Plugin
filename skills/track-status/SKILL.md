---
name: track-status
description: Update the status of an existing outreach log row. Use when a contact replies, books a meeting, the meeting happens, the conversation stalls, or the opportunity closes. Records state-change date without rewriting history. The recommendation engine reads these statuses to enforce anti-loop discovery.
disable-model-invocation: false
allowed-tools: Read, Edit, Bash(test *), Bash(date *), Bash(grep *)
---

# Track Status

State-change updates on outreach log rows. Append-only spirit: the date columns shift, but the original `date` column (when the outreach happened) does not.

## Inputs

`$ARGUMENTS`:
- **Required identifier** — one of:
  - `--row=<n>` — 1-based row index in `outreach.md`.
  - `--company=<slug> --contact=<name>` — disambiguates if multiple outreach events to same person.
  - `--latest=<company-slug>` — picks the most recent outreach row for that company.
- **Required**: `--status=<sent|replied|meeting-booked|meeting-done|stalled|closed-won|closed-lost>`
- Optional: `--note=<freeform>` — appended to existing notes.
- Optional: `--next-action=<freeform>` — overwrites next-action column.

### Examples

```
/career:outreach status --latest=snowglobe --status=replied
/career:outreach status --latest=acme --status=meeting-booked --next-action="prep deck for Tue 2pm"
/career:outreach status --row=12 --status=closed-lost --note="passed on YOE mismatch"
```

## Procedure

### 1. Resolve config + path

`~/.config/career-os/config.json` → `WORKING_FOLDER`.
File: `${WORKING_FOLDER}/crm/outreach.md`.

### 2. Locate the row

By the identifier mode used. If `--latest=<slug>` returns zero or multiple ambiguous rows, list candidates and ask.

### 3. Validate transition

Allowed transitions:

```
sent          → replied | stalled | closed-lost
replied       → meeting-booked | stalled | closed-lost
meeting-booked → meeting-done | stalled | closed-lost
meeting-done  → meeting-booked | stalled | closed-won | closed-lost
stalled       → replied | meeting-booked | closed-lost | closed-won
closed-won    → (terminal — warn, ask confirm)
closed-lost   → (terminal — warn, ask confirm)
```

If the requested transition is unusual (e.g. `sent` → `closed-won` directly), warn and confirm before applying.

### 4. Update row

In place. Specifically:
- `status` column → new value.
- `notes` column → append `[YYYY-MM-DD: <new-status>] <note>` (if note provided), else `[YYYY-MM-DD: <new-status>]`.
- `next-action` column → overwrite if `--next-action` provided.

The original `date` column (first-touch date) is **not** modified. The `last-touch` concept lives in `companies.md`, not in the per-event `outreach.md`.

### 5. Update companies.md

In `${WORKING_FOLDER}/crm/companies.md` for the matching slug:
- `last-touch` → today.
- `status`:
  - `replied | meeting-booked | meeting-done` → `active`
  - `stalled` → `paused`
  - `closed-won` → `active` (the engagement is alive)
  - `closed-lost` → `passed`
  - `sent` → `engaged` (no change if already)

### 6. Companion delegation

If status is `meeting-booked` and `schedule-manager` companion is installed, offer:

```
Add to calendar?
  /schedule-manager:create-event
```

Don't auto-invoke; the user may want to set agenda/attendees themselves.

If status is `meeting-done`, offer to draft a follow-up via `draft-outreach <slug> --template=warm-followup`.

### 7. Print confirmation

```
updated: row <n> | <company-slug> | <contact> | <old-status> → <new-status>
notes:   <appended fragment>
```

## Guardrails

- The original outreach `date` column is immutable.
- State transitions follow the table above; unusual ones require explicit confirm.
- Terminal states (`closed-won`, `closed-lost`) require confirmation to modify.
- Never collapse rows — even if multiple updates happen the same day, each shows up as a separate `[YYYY-MM-DD: <status>]` fragment in notes.

## Failure modes

- **Multiple matching rows from `--latest`** → list, ask.
- **Row not found** → bail; suggest `cat crm/outreach.md | head` to inspect.
- **Invalid status enum** → list valid values.

## Idempotency

- Updating to the same status is allowed and adds another `[date: status]` fragment (useful for "still stalled" check-ins).
- Re-running with the same status + note is allowed; deduplication is the user's responsibility.

---
name: log-outreach
description: Manually log an outreach that was sent outside the plugin (e.g. a LinkedIn message, an in-person conversation, an email sent from a phone). Appends a row to crm/outreach.md without invoking any send. Use this to keep the outreach log honest as the spine of the recommendation engine.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), Bash(date *)
---

# Log Outreach

Manual entry into the outreach log. The send-outreach skill calls this implicitly on send; this skill exists for everything that didn't go through the plugin.

## Inputs

`$ARGUMENTS` — interactive if missing, otherwise:
- `--company=<name-or-slug>`
- `--contact=<name>`
- `--channel=<email|linkedin|intro|event|referral|other>` (default `email`)
- `--template=<template-name-or-freeform>`
- `--status=<sent|replied|meeting-booked|meeting-done|stalled|closed-won|closed-lost>` (default `sent`)
- `--date=<YYYY-MM-DD>` (default today)
- `--next-action=<freeform>`
- `--notes=<freeform>`

## Procedure

### 1. Resolve config + path

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. Confirm `${WORKING_FOLDER}/crm/outreach.md` exists; if not, copy from the plugin template.

### 2. Gather missing fields

Use `AskUserQuestion` for any missing required fields:
- company (free text → resolve to slug)
- contact (free text)
- channel (enum)
- template (free text — for off-platform outreach the user may write a custom phrase like "linkedin DM, custom" — that's fine)
- status (enum, default `sent`)
- next-action (free text)

Date defaults to today. Notes are optional.

### 3. Resolve company slug

If `<company>` is in `crm/companies.md`, use that slug. Otherwise:
- Compute slug from name (kebab-case).
- Append a new row to `companies.md` with status `engaged`, last-touch today.

### 4. Append row

To `${WORKING_FOLDER}/crm/outreach.md`:

```
| <date> | <company-slug> | <contact> | <channel> | <template> | <status> | <next-action> | <notes> |
```

Append at end of the table; never rewrite history.

### 5. Update CRM company row

Set `crm/companies.md` row's `last-touch` to the date.

### 6. Print confirmation

```
logged: <company-slug> / <contact> / <channel> / <status>
file:   crm/outreach.md
```

## Guardrails

- Append-only. Never edit existing rows. (Use `track-status` for state changes.)
- Slug discipline — same slug across companies.md, outreach.md, and research/ dir.
- Don't infer status. If user said "I emailed them yesterday" but didn't say replied, default to `sent`.

## Failure modes

- **`crm/outreach.md` missing** → restore from plugin template before appending.

## Idempotency

- Re-running with same args creates a duplicate row (this is correct — multiple outreach attempts to same contact are different events).
- If the user wants to *update* an existing row, use `track-status` instead.

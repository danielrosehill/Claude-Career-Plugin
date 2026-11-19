---
name: send-outreach
description: Send a previously-drafted outreach email. Reads a draft file from drafts/, checks it for blockers, then sends it with no review step (unless the user asked to review first), delegating to the email-skills companion plugin (personal or business sender per draft frontmatter). On send, updates the draft frontmatter to status=sent and appends a row to crm/outreach.md. Holds only when the user asked to review, or a blocker applies.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), Bash(date *)
---

# Send Outreach

Sends a draft. Sends by default: no confirmation prompt. Always logs to `crm/outreach.md` on send.

## Inputs

`$ARGUMENTS`:
- **Required**: `<draft-path>` — path (relative to `WORKING_FOLDER` or absolute) to the draft file. Usually `drafts/<YYYY-MM-DD>-<slug>-<contact>.md`.
- Optional: `--force` — bypass the "no recent outreach to this contact" anti-loop check.

## Procedure

### 1. Resolve config + read draft

Read `${CAREER_DATA_DIR}/config.json`:
- `WORKING_FOLDER`
- `PREFERRED_EMAIL_SENDER`, `SECONDARY_EMAIL_SENDER`
- `companions.email` — must be `true`. If `false`, bail with instruction to run `/career:install-companion-plugins`.

Read the draft file. Parse frontmatter. Validate:
- `status: draft` (refuse to re-send something already `sent`).
- `contact.email` present and looks like an email.
- `sender: personal | business` resolved.
- Body is non-empty.

### 2. Anti-loop check

Read `${WORKING_FOLDER}/crm/outreach.md`. If a row exists for the same contact email with `last-touch < 14 days` and status `sent | replied | meeting-booked`: stop and require `--force`.

### 3. Pick the email skill

| sender field in draft | delegate to |
| --- | --- |
| `personal` | `email-skills:send-personal-email` |
| `business` | `email-skills:send-business-email` |

If draft `sender_email` is set, pass it through; otherwise let the email skill resolve from its own config.

### 4. Pre-send check (no confirmation prompt)

Default: send. When the user asks for an email, or a task calls for one, and what to say is clear, send it: applications, cold emails, pitches and formal emails included. No review step, no yes/no/edit menu, no Gmail draft left behind. Hold for review only when the user asked ("draft it", "show me first", "let me review", "don't send").

Real blockers still stop the send: no verified address, a fact only the user can supply, a leftover placeholder (`{{...}}`), an attachment (e.g. a CV) that cannot be found. Name the blocker and stop.

Hebrew is the one hard check: every Hebrew paragraph in a raw `<div dir="rtl" style="text-align:right;">` (never entity-escaped), the Latin signature in a `dir="ltr"` block. After sending, read the sent message back, check the dir attributes survived and numbers, dates, phones and URLs read in order; if broken, fix and resend.

### 5. Send

Invoke the chosen email skill with `to`, `subject`, `body`. Capture the return value (message-id if available). Report to the user in one line: recipient, subject, message id.

### 6. Update draft frontmatter

Set:
- `status: sent`
- `sent_at: <ISO datetime>`
- `message_id: <id>` (if returned)

### 7. Append to outreach log

Append one row to `${WORKING_FOLDER}/crm/outreach.md`:

```
| <YYYY-MM-DD> | <company> | <contact-name> | email | <template> | sent | follow up <YYYY-MM-DD>+7 | <link to draft> |
```

Notes column: relative path to the draft file so the row links back to the full message.

### 8. Update CRM company row

In `${WORKING_FOLDER}/crm/companies.md`, set the company row's `status` to `engaged` (unless already `active` / `paused` — leave those) and `last-touch` to today.

### 9. Print summary

```
sent: <name> <email>
template: <template>
follow-up due: <date+7>
log:  crm/outreach.md (row appended)
```

## Guardrails

- **Sends by default**; holds only when the user asked to review or a blocker (step 4) applies.
- **Never re-sends** a draft already marked `sent`.
- **Anti-loop** by default; `--force` opt-out.
- **No silent edits.** If the user wants to change the draft, they re-edit and re-run; this skill does not modify body content.

## Failure modes

- **`email-skills` not installed** → bail with install instructions.
- **Email sender misconfigured** (no `from` resolved) → bail; print which env / config the email skill expects.
- **Anti-loop blocks** → show the existing row, recommend `track-status` to update first, or `--force` if intentional re-touch.
- **Email skill returns error** → leave draft as `draft`, do not log to outreach.md, surface the error verbatim.

## Idempotency

- A `sent` draft cannot be re-sent without manually flipping `status` back to `draft` (deliberate friction).
- Re-running on the same `draft` after a send error retries cleanly.

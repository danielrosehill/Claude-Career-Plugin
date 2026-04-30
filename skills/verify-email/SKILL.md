---
name: verify-email
description: Validate a single email address via Hunter Email-Verifier. Use when the user has an address (handed to them, scraped, guessed) and wants a deliverability + confidence read before sending outreach. Returns Hunter's verdict (deliverable / risky / undeliverable / unknown), score, and the disposable / role / accept-all flags. Does not write to the CRM unless asked.
disable-model-invocation: false
allowed-tools: Read, Edit, mcp__jungle-personal__hunter__Email-Verifier
---

# Verify Email

Wraps Hunter's `Email-Verifier`. One address in, one verdict out.

## Inputs

`$ARGUMENTS`:
- **Required**: `<email>` — first positional.
- Optional: `--update-crm` — if the email matches a row in `${CAREER_WORKSPACE}/crm/contacts.md`, update the `verified` and `last-verified` columns.

## Procedure

### 1. Sanity check input

If `<email>` is malformed (no `@`, no TLD), bail with a one-line error — don't spend a Hunter credit on garbage.

### 2. Call Hunter

Call `Email-Verifier` with the address. Capture: `result`, `score`, `disposable`, `webmail`, `mx_records`, `smtp_check`, `accept_all`, `block`.

### 3. Render

Compact summary:

```
email:        jane.doe@acme.com
verdict:      deliverable
score:        92
flags:        webmail=no, accept_all=no, disposable=no
mx / smtp:    yes / yes
```

Add a one-line interpretation:
- `deliverable` + score ≥ 80 → safe to send.
- `risky` or score 50–79 → send only if you have a fallback channel.
- `undeliverable` / `block` → do not send.
- `accept_all=yes` → Hunter can't truly verify; treat as risky.

### 4. Optional: update CRM row

If `--update-crm`:

Load `${CAREER_WORKSPACE}/crm/contacts.md` (resolve `CAREER_WORKSPACE` per the standard order: env var → `config.json` `WORKING_FOLDER` → `$PWD`). Find the row whose `email` matches. Update `verified` (yes/no/risky) and append `last-verified=<YYYY-MM-DD>` to the notes column. If no row matches, say so — don't insert.

## Failure modes

- **Hunter MCP not loaded** → bail with: enable the `hunter` MCP via this plugin's `.mcp.json` (set `HUNTER_API_KEY`).
- **429 rate limit** → surface verbatim, stop.
- **Address not in standard format** → bail before calling Hunter.

## Idempotency

Re-running on the same address re-hits Hunter (results can change as Hunter's index updates). `--update-crm` is overwrite-in-place, never duplicates rows.

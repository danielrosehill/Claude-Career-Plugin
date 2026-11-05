---
name: verify-email
description: Validate a single email address via the Hunter email-verifier API. Use when the user has an address (handed to them, scraped, guessed) and wants a deliverability + confidence read before sending outreach. Returns Hunter's verdict (deliverable / risky / undeliverable / unknown), score, and the disposable / role / accept-all flags. Does not write to the CRM unless asked.
disable-model-invocation: false
allowed-tools: Read, Edit, Bash(with-secrets *)
---

# Verify Email

Wraps Hunter's `email-verifier` endpoint. One address in, one verdict out.

Route: Hunter REST API v2 with `curl` (no MCP). Key `HUNTER_API_KEY` is in Infisical; call as `with-secrets bash -c 'curl -s -G "https://api.hunter.io/v2/<endpoint>" --data-urlencode "api_key=$HUNTER_API_KEY" --data-urlencode "k=v"'`. Never print the key or write it to a file. Payload is under `data`, errors under `errors`. The account is on the Free plan (50 searches and 100 verifications a month), so keep calls to what is needed. Check the balance with `GET /v2/account` (free).


## Inputs

`$ARGUMENTS`:
- **Required**: `<email>` — first positional.
- Optional: `--update-crm` — if the email matches a row in `${CAREER_WORKSPACE}/crm/contacts.md`, update the `verified` and `last-verified` columns.

## Procedure

### 1. Sanity check input

If `<email>` is malformed (no `@`, no TLD), bail with a one-line error — don't spend a Hunter credit on garbage.

### 2. Call Hunter

`GET /v2/email-verifier?email=<address>`. Capture: `status` (`valid` / `invalid` / `accept_all` / `webmail` / `disposable` / `unknown`), `result` (older field: `deliverable` / `risky` / `undeliverable`), `score`, `disposable`, `webmail`, `mx_records`, `smtp_check`, `accept_all`, `block`.

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

- **`HUNTER_API_KEY` missing or 401 (`authentication_failed`)** → bail with: the key lives in Infisical; run the call under `with-secrets`.
- **429 rate limit** → surface verbatim, stop.
- **Address not in standard format** → bail before calling Hunter.

## Idempotency

Re-running on the same address re-hits Hunter (results can change as Hunter's index updates). `--update-crm` is overwrite-in-place, never duplicates rows.

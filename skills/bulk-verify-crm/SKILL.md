---
name: bulk-verify-crm
description: Sweep all (or a filtered subset of) email addresses in the workspace CRM and re-verify them via Hunter Email-Verifier. Use periodically — people change jobs, addresses go stale, accept-all status changes. Updates the `verified` and `last-verified` columns in `crm/contacts.md` and prints a summary of what flipped.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), mcp__jungle-personal__hunter__Email-Verifier
---

# Bulk Verify CRM

Periodic sweep of `${CAREER_WORKSPACE}/crm/contacts.md`. Throttle-aware — Hunter's verifier consumes one credit per address.

## Inputs

`$ARGUMENTS`:
- Optional: `--max=<N>` — cap the number of addresses verified this run (default 50).
- Optional: `--stale-days=<N>` — only re-verify rows whose `last-verified` is older than N days (default 90). Rows never verified are always included.
- Optional: `--filter=<expression>` — restrict to rows matching a substring of any column (e.g. `--filter=acme` or `--filter=lens=employer`).
- Optional: `--dry-run` — list what would be verified, don't call Hunter.

## Procedure

### 1. Resolve workspace + load CRM

Resolve `CAREER_WORKSPACE` (env → `config.json` `WORKING_FOLDER` → `$PWD`). Load `${CAREER_WORKSPACE}/crm/contacts.md`. If absent, bail.

### 2. Build the work list

For each row:
- Skip if no email or email is malformed.
- Skip if `last-verified` is within `--stale-days`.
- Skip if `--filter` is set and no column substring matches.
- Stop adding once `--max` is reached.

Print the work list count and total Hunter credits this will consume. If `--dry-run`, stop here.

### 3. Confirm before spending

If the work list is > 10 rows, ask the user to confirm — bulk verification has real cost.

### 4. Verify

For each address in the work list, call `Email-Verifier`. Capture `result` and `score`. If a 429 comes back, stop the sweep, write everything verified so far, surface the rate-limit message.

### 5. Update CRM

For each verified row, update:
- `verified` column → `yes` / `no` / `risky` (per the verdict mapping in `verify-email`).
- `last-verified` → today's date (DD/MM/YY).
- If verdict flipped from `yes` → `no`/`risky`, append `flipped=<YYYY-MM-DD>` to notes so the user can spot drift.

### 6. Summary

Print:

```
sweep complete:
  verified:    37
  unchanged:   29
  flipped:     6
    - jane@acme.com    yes -> no
    - sam@beta.io      yes -> risky
    ...
  errors:      2
```

Suggest next actions: re-run `find-contact` for any company whose primary contact flipped to `no` / `risky`.

## Guardrails

- **Never call Hunter without confirmation for > 10 rows.** Credit cost is real.
- **Don't auto-delete contact rows** even if Hunter says undeliverable — the user may still want the row for record-keeping. Just flip `verified`.
- **Respect `--max`.** Default 50 is intentionally conservative.

## Idempotency

Re-running picks up where the last run left off because rows verified within `--stale-days` are skipped. Safe to schedule (e.g. once a quarter via the `schedule` skill).

## Failure modes

- **No `crm/contacts.md` in the workspace** — bail with: "no CRM yet — run `find-contact <company> --append-to-crm` first."
- **Hunter MCP not loaded** — bail; do not partial-run.
- **Rate limit mid-sweep** — write what you have, report the cutoff, exit cleanly.

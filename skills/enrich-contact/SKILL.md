---
name: enrich-contact
description: Enrich an existing CRM contact with Hunter combined enrichment — pulls role, seniority, department, social handles, and the contact's company facts in one call. Use to flesh out a thinly-populated CRM row before drafting outreach or prepping for a meeting. Operates on an email address from the CRM; updates the row in place.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(with-secrets *)
---

# Enrich Contact

Wraps Hunter's combined enrichment endpoint. One email in, person + company facts out, optionally written back to the CRM.

Route: Hunter REST API v2 with `curl` (no MCP). Key `HUNTER_API_KEY` is in Infisical; call as `with-secrets bash -c 'curl -s -G "https://api.hunter.io/v2/<endpoint>" --data-urlencode "api_key=$HUNTER_API_KEY" --data-urlencode "k=v"'`. Never print the key or write it to a file. Payload is under `data`, errors under `errors`. The account is on the Free plan (50 searches and 100 verifications a month), so keep calls to what is needed. Check the balance with `GET /v2/account` (free).

The enrichment endpoints (`/v2/combined/find`, `/v2/people/find`, `/v2/companies/find`) are from the Hunter docs and have not been tested on this account; the Free plan may not include them. On a 401/403/plan error, say so and stop.

## Inputs

`$ARGUMENTS`:
- **Required**: `<email>` — first positional. Must already exist as a row in `${CAREER_WORKSPACE}/crm/contacts.md` unless `--ad-hoc` is passed.
- Optional: `--ad-hoc` — skip the CRM-row precondition; just print enrichment.
- Optional: `--no-write` — print the enrichment but don't update the CRM row.

## Procedure

### 1. Resolve workspace + load row

Resolve `CAREER_WORKSPACE` (env → `config.json` `WORKING_FOLDER` → `$PWD`).

Load `${CAREER_WORKSPACE}/crm/contacts.md`. Find the row whose `email` matches `<email>`. If none and `--ad-hoc` not set, bail with: "no contact row for `<email>` — add one with `find-contact --append-to-crm` first, or pass `--ad-hoc`."

### 2. Call combined enrichment

`GET /v2/combined/find?email=<email>`. Person data is under `data.person`, company data under `data.company`.

Capture from the response:
- Person: `full_name`, `position`, `seniority`, `department`, `linkedin`, `twitter`.
- Company: `name`, `domain`, `industry`, `size`, `country`, `description`.

If only person OR company data comes back, that's fine — render what's there.

### 3. Render

Two-block summary:

```
person:
  name:       Jane Doe
  position:   Head of Platform
  seniority:  senior
  department: engineering
  linkedin:   in/janedoe

company:
  name:       Acme Corp
  domain:     acme.com
  industry:   B2B SaaS
  size:       51-200
  country:    US
  one-liner:  ... (description, trimmed)
```

### 4. Update CRM row

Unless `--no-write` or `--ad-hoc`:

- Update the contact row's `role` column from `position` if it was empty or "—".
- Append `seniority=<value>, dept=<value>` to the `notes` column if missing.
- Touch `last-touch` to today's date (DD/MM/YY per Daniel's preference).

If the company row in `${CAREER_WORKSPACE}/crm/companies.md` is sparse, also fill `industry` and `size` columns (don't overwrite existing non-empty values without asking).

## Failure modes

- **Hunter returns 404 / no enrichment** — say so; leave the CRM row untouched.
- **`HUNTER_API_KEY` missing or 401 (`authentication_failed`)** — same bail as the other Hunter skills.
- **Multiple CRM rows match the email** — print them and ask which one to update.

## Idempotency

Safe to re-run — only fills empty fields by default. Prompts before overwriting non-empty values.

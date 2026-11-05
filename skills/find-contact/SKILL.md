---
name: find-contact
description: Find the right person to contact at a target company using Hunter.io. Use when the user needs decision-maker / hiring-manager / founder contact details before drafting an outreach. Returns ranked candidates with confidence scores; optionally appends to the workspace CRM. Calls the Hunter REST API directly.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), Bash(with-secrets *)
---

# Find Contact

Hunter.io wrapper. Finds people at a target company and returns ranked candidates the user can pass into `draft-outreach`.

Route: Hunter REST API v2 with `curl` (no MCP). Key `HUNTER_API_KEY` is in Infisical; call as `with-secrets bash -c 'curl -s -G "https://api.hunter.io/v2/<endpoint>" --data-urlencode "api_key=$HUNTER_API_KEY" --data-urlencode "k=v"'`. Never print the key or write it to a file. Payload is under `data`, errors under `errors`. The account is on the Free plan (50 searches and 100 verifications a month), so keep calls to what is needed. Check the balance with `GET /v2/account` (free).


## Inputs

`$ARGUMENTS`:
- **Required**: `<company>` — name or domain. First positional.
- Optional: `--role=<filter>` — e.g. `engineering manager`, `founder`, `head of product`. Filters Hunter results by department/seniority.
- Optional: `--n=<count>` (default 5) — max candidates to return.
- Optional: `--append-to-crm` — write rows into `crm/companies.md`.
- Optional: `--lens=<employer|client|partner>` — sets the CRM row's lens column (default `employer`).

### Examples

```
/career:outreach find acme.com
/career:outreach find snowglobe.ai --role="head of platform" --n=3
/career:outreach find myrofish --role=founder --append-to-crm
```

## Procedure

### 1. Resolve config

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. If `companions.email` is `false`, warn the user that `send-outreach` won't work yet — `find-contact` can still proceed.

### 2. Resolve domain

If `<company>` looks like a domain (`*.tld`), use directly.
Otherwise, resolve the domain from the company's site or a web search (Hunter's endpoints are keyed by domain or email, not by name).

If Hunter returns multiple candidate domains, ask the user which one.

### 3. Pull contacts

`GET /v2/domain-search?domain=<domain>&limit=<n>` (add `type=personal`, and `department=` / `seniority=` from `--role`). Addresses are in `data.emails[]` (`value`, `first_name`, `last_name`, `position`, `confidence`, `sources`). Run `GET /v2/email-count?domain=` first (free); if `data.total` is 0, stop.

Limit to `--n` results. Hunter returns confidence scores — pass through.

### 4. Optionally enrich + verify

For the top `--n` results:
- `GET /v2/email-verifier?email=` to check deliverability — adds a `verified` flag.
- Skip enrichment unless the user asks; it costs additional credits.

### 5. Render

Output a markdown table to the user:

```
| rank | name | role | email | confidence | verified | source |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | ... | ... | ... | 92 | yes | hunter |
```

Include a "next step" hint:

```
To draft outreach to one of these:
  /career:outreach draft <company> --contact-rank=1 --template=cold-pitch
```

### 6. Append to CRM if requested

If `--append-to-crm`:

For each contact, ensure `${WORKING_FOLDER}/crm/companies.md` has a row for the company (lens, status `watching`, last-touch today). The contacts table itself lives in the brief output; the CRM row is per-company, not per-contact.

For each contact, ensure `${WORKING_FOLDER}/crm/contacts.md` exists; create with header if not:

```
| company-slug | name | role | email | confidence | verified | last-touch | notes |
```

Then append a row per contact.

## Guardrails

- **Never auto-send anything.** This skill produces a list. Sending is `send-outreach` with explicit confirm.
- Hunter rate-limits — if a 429 comes back, surface verbatim and stop.
- Don't synthesise email addresses from name + domain heuristics. Only return what Hunter returns.
- If Hunter returns nothing, say so; don't fall back to guessing.

## Failure modes

- **`HUNTER_API_KEY` missing or 401 (`authentication_failed`)** — bail with a one-line instruction: the key lives in Infisical; run the call under `with-secrets`.
- **Domain resolution ambiguous** — present candidates, ask the user.
- **Zero results** — print "no contacts returned for <domain>"; suggest broadening role filter.

## Idempotency

- Re-running with same args → same Hunter call (results may differ if Hunter's index changed).
- `--append-to-crm` is upsert: existing rows are updated by `name + email`, not duplicated.

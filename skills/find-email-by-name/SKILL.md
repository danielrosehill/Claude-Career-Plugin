---
name: find-email-by-name
description: Given a person's name and their company (name or domain), return Hunter's best guess at their email address with a confidence score. Use when the user already knows who they want to reach and just needs the address. Differs from find-contact (which lists candidates at a company) — this is targeted at a single named person.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(with-secrets *)
---

# Find Email By Name

Wraps Hunter's `email-finder` endpoint. Targeted lookup for a single person.

Route: Hunter REST API v2 with `curl` (no MCP). Key `HUNTER_API_KEY` is in Infisical; call as `with-secrets bash -c 'curl -s -G "https://api.hunter.io/v2/<endpoint>" --data-urlencode "api_key=$HUNTER_API_KEY" --data-urlencode "k=v"'`. Never print the key or write it to a file. Payload is under `data`, errors under `errors`. The account is on the Free plan (50 searches and 100 verifications a month), so keep calls to what is needed. Check the balance with `GET /v2/account` (free).


## Inputs

`$ARGUMENTS`:
- **Required**: `<full-name>` and `<company-or-domain>`. Supplied as either two positionals or `--name="Jane Doe" --company=acme.com`.
- Optional: `--verify` — also run `GET /v2/email-verifier` on the returned address.
- Optional: `--append-to-crm` — append a row to `${CAREER_WORKSPACE}/crm/contacts.md`.
- Optional: `--lens=<employer|client|partner>` — for the CRM row.

### Examples

```
/career:outreach find-email "Jane Doe" acme.com
/career:outreach find-email --name="Sam Patel" --company="Snowglobe AI" --verify
```

## Procedure

### 1. Resolve domain

If `<company-or-domain>` looks like a domain, use it directly. Otherwise resolve the domain from the company's site or a web search (Hunter's endpoints are keyed by domain, not by name). If several candidates come back, ask the user which.

### 2. Split the name

Split on the first space → `first_name`, `last_name`. If the user supplied only one token, ask for the other before calling Hunter — the finder needs both.

### 3. Call email-finder

`GET /v2/email-finder?first_name=<first>&last_name=<last>&domain=<domain>`. Capture, under `data`: `email`, `score`, `sources` (count + URLs), `verification.status`.

### 4. Optional verify

If `--verify`, call `GET /v2/email-verifier` on the returned address. Merge the verifier's `result` and `score` into the output.

### 5. Render

```
person:    Jane Doe
domain:    acme.com
email:     jane.doe@acme.com
score:     94
sources:   3 ( linkedin.com, acme.com/team, ... )
verified:  deliverable (92)   ← only if --verify
```

Hint the next step:

```
/career:outreach draft acme.com --to=jane.doe@acme.com --template=cold-pitch
```

### 6. Optional CRM append

If `--append-to-crm`, ensure `${CAREER_WORKSPACE}/crm/contacts.md` exists with the standard header, then append a row. Upsert by `email` — don't duplicate.

## Failure modes

- **Hunter returns no email** — say so explicitly. Don't synthesise from a `firstname.lastname@domain` template; those guesses are exactly what `find-contact` is supposed to avoid.
- **Score below 50** — return it, but flag as low-confidence.
- **`HUNTER_API_KEY` missing or 401 (`authentication_failed`)** — bail with the same message as `find-contact` / `verify-email`.

## When to use this vs find-contact

- `find-contact` — "who at acme should I contact?" (returns several candidates).
- `find-email-by-name` — "I know I want Jane Doe; what's her email?" (returns one address).

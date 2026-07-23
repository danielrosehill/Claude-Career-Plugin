---
name: find-contact
description: Find the right person to contact at a target company using Hunter.io. Use when the user needs decision-maker / hiring-manager / founder contact details before drafting an outreach. Returns ranked candidates with confidence scores; optionally appends to the workspace CRM. Hunter MCP is required.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), mcp__gateway__hunter__Domain-Search, mcp__gateway__hunter__Email-Finder, mcp__gateway__hunter__Email-Verifier, mcp__gateway__hunter__Combined-Enrichment, mcp__gateway__hunter__Company-Enrichment
---

# Find Contact

Hunter.io wrapper. Finds people at a target company and returns ranked candidates the user can pass into `draft-outreach`.

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
Otherwise, use Hunter `Company-Enrichment` (or `Combined-Enrichment` if a contact name is also implied) to resolve company name → primary domain.

If Hunter returns multiple candidate domains, ask the user which one.

### 3. Pull contacts

`Domain-Search` with the resolved domain. Apply role / department filter from `--role`.

Limit to `--n` results. Hunter returns confidence scores — pass through.

### 4. Optionally enrich + verify

For the top `--n` results:
- `Email-Verifier` to check deliverability — adds a `verified` flag.
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
- Don't synthesise email addresses from name + domain heuristics. Only return what Hunter / Email-Finder verifies.
- If Hunter returns nothing, say so; don't fall back to guessing.

## Failure modes

- **Hunter MCP not available** — bail with one-line instruction: install `hunter` MCP via the personal MCP catalog or `op-vault`.
- **Domain resolution ambiguous** — present candidates, ask the user.
- **Zero results** — print "no contacts returned for <domain>"; suggest broadening role filter.

## Idempotency

- Re-running with same args → same Hunter call (results may differ if Hunter's index changed).
- `--append-to-crm` is upsert: existing rows are updated by `name + email`, not duplicated.

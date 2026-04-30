---
name: enrich-contact
description: Enrich an existing CRM contact with Hunter Combined-Enrichment — pulls role, seniority, department, social handles, and the contact's company facts in one call. Use to flesh out a thinly-populated CRM row before drafting outreach or prepping for a meeting. Operates on an email address from the CRM; updates the row in place.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, mcp__jungle-personal__hunter__Combined-Enrichment, mcp__jungle-personal__hunter__Company-Enrichment
---

# Enrich Contact

Wraps Hunter's `Combined-Enrichment`. One email in, person + company facts out, optionally written back to the CRM.

## Inputs

`$ARGUMENTS`:
- **Required**: `<email>` — first positional. Must already exist as a row in `${CAREER_WORKSPACE}/crm/contacts.md` unless `--ad-hoc` is passed.
- Optional: `--ad-hoc` — skip the CRM-row precondition; just print enrichment.
- Optional: `--no-write` — print the enrichment but don't update the CRM row.

## Procedure

### 1. Resolve workspace + load row

Resolve `CAREER_WORKSPACE` (env → `config.json` `WORKING_FOLDER` → `$PWD`).

Load `${CAREER_WORKSPACE}/crm/contacts.md`. Find the row whose `email` matches `<email>`. If none and `--ad-hoc` not set, bail with: "no contact row for `<email>` — add one with `find-contact --append-to-crm` first, or pass `--ad-hoc`."

### 2. Call Combined-Enrichment

```
Combined-Enrichment { email }
```

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
- **Hunter MCP not loaded** — same bail as the other Hunter skills.
- **Multiple CRM rows match the email** — print them and ask which one to update.

## Idempotency

Safe to re-run — only fills empty fields by default. Prompts before overwriting non-empty values.

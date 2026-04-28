---
name: crm-adapter-attio
description: Two-way sync between markdown CRM and Attio. Same shape as crm-adapter-airtable but mapped to Attio's object model (Companies, People, Deals, Notes). Markdown stays canonical; Attio is a working surface. Conflicts surfaced, never auto-merged.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(test *), Bash(date *)
---

# CRM Adapter — Attio

Same contract as `crm-adapter-airtable`, but for Attio. Use this when the user is already living in Attio for personal CRM (common for indie consultants).

Markdown remains the source of truth.

## Inputs

`$ARGUMENTS`:
- **Required**: `--direction=<push|pull|two-way>`.
- Optional: `--object=<companies|people|deals|all>` — default `all`.
- Optional: `--dry-run`.

## Prerequisites

- Attio MCP or API token configured.
- Config:
  - `CRM_ADAPTER=attio`
  - `ATTIO_WORKSPACE_ID`
  - `ATTIO_OBJECT_COMPANIES`, `ATTIO_OBJECT_PEOPLE`, `ATTIO_OBJECT_DEALS` — Attio object slugs (default `companies`, `people`, `deals`).

## Schema mapping

`crm/companies.md` ↔ Attio `Companies`:

| markdown column | attio attribute |
| --- | --- |
| slug | `domains.domain` (or custom `Slug` text attr) |
| name | `name` |
| domain | `domains` |
| status | `categories` (or custom select) |
| size | custom `Size` |
| hq | `primary_location` |
| notes | linked `Notes` |
| last-touch | custom `Last Touch` (date) |

`crm/outreach.md` ↔ a custom `Outreach` object (created on first run if missing) or treated as Attio `Notes` against the linked Company. Default: custom `Outreach` object.

`crm/opportunities.md` ↔ Attio `Deals`.

People (recipients) are upserted into Attio `People` with the company link; markdown rows reference recipient name + email only.

## Procedure

Identical to `crm-adapter-airtable` (load both sides → diff → apply per direction → surface conflicts → manifest at `.attio-sync.json`), differing only in:

- API calls go through Attio MCP/REST.
- `slug` is keyed via the company's primary domain by default; configurable via `ATTIO_SLUG_ATTR`.
- Outreach object: if `ATTIO_OBJECT_OUTREACH` not present, propose creating a custom object on first run (asks the user).

## Conflict handling

Same as airtable adapter — write to `crm/.conflicts-<date>.md`, never auto-resolve.

## Print summary

```
direction: <push|pull|two-way>
pushed: <n>  pulled: <n>  unchanged: <n>  conflicts: <n>
manifest: .attio-sync.json
custom-objects-created: <list, if any>
```

## Guardrails

- Markdown canonical.
- Don't create a custom Attio object without asking — Attio object schema is sticky.
- If a user has a non-default object schema, the user's mapping in config wins.

## Failure modes

- Missing Attio credentials → bail with setup steps.
- Required custom attribute missing → ask the user whether to create it (one prompt per attribute).
- API rate limit → checkpoint and exit cleanly.

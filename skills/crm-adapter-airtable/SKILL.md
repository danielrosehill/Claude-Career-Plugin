---
name: crm-adapter-airtable
description: Two-way sync between markdown CRM (crm/companies.md, crm/opportunities.md, crm/outreach.md) and Airtable. Reads CRM_ADAPTER=airtable. Markdown stays the source of truth — Airtable is a mirror with a diff back. Conflicts surfaced for user resolution, never auto-merged.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(test *), Bash(date *)
---

# CRM Adapter — Airtable

Markdown CRM works fine until you want a sortable view, mobile access, or a teammate. This adapter mirrors the markdown rows into Airtable and pulls back changes the user made there.

Markdown remains the source of truth. Airtable is a working surface.

## Inputs

`$ARGUMENTS`:
- **Required**: `--direction=<push|pull|two-way>`.
- Optional: `--table=<companies|opportunities|outreach|all>` — default `all`.
- Optional: `--dry-run` — print the diff plan, no writes.

## Prerequisites

- Airtable MCP installed (or PAT-configured CLI).
- Config keys:
  - `CRM_ADAPTER=airtable`
  - `AIRTABLE_BASE_ID`
  - `AIRTABLE_TABLE_COMPANIES`, `AIRTABLE_TABLE_OPPORTUNITIES`, `AIRTABLE_TABLE_OUTREACH`

If config missing, bail with the keys to set.

## Schema mapping

`crm/companies.md` rows ↔ Airtable `Companies`:

| markdown column | airtable field |
| --- | --- |
| slug (id) | `Slug` (primary) |
| name | `Name` |
| domain | `Domain` |
| status | `Status` (single select) |
| size | `Size` |
| hq | `HQ` |
| notes | `Notes` (long text) |
| last-touch | `Last Touch` (date) |

`crm/opportunities.md` ↔ `Opportunities`: slug, company-slug (linked record), role/engagement, stage, amount, opened, closed, notes.

`crm/outreach.md` ↔ `Outreach`: id, company-slug (linked), recipient, channel, sent-date, status, follow-up-due, draft-path, notes.

Linked records (company-slug → Companies) resolved by slug.

## Procedure

### 1. Load both sides

- Parse markdown CRM tables into rows keyed by slug.
- Pull current Airtable records via MCP into the same shape.

### 2. Diff

For each row, classify:
- `local-only` — in markdown, not in Airtable.
- `remote-only` — in Airtable, not in markdown.
- `match` — same in both.
- `conflict` — same slug, divergent values (and both modified since last sync).

Use `${WORKING_FOLDER}/.airtable-sync.json` to track last-sync timestamps per row, so a one-sided edit is *not* a conflict.

### 3. Apply

- `--direction=push`: write `local-only` and updated-local rows to Airtable. Don't touch remote-only or conflicts.
- `--direction=pull`: write `remote-only` and updated-remote rows to markdown. Don't touch local-only or conflicts.
- `--direction=two-way`: push local, pull remote, surface conflicts.

For markdown writes: edit the table cells, preserving file structure. Append a `**Last synced:** YYYY-MM-DDTHH:MM` line under the table header.

For Airtable writes: use MCP create/update by `Slug`.

### 4. Conflict handling

For each conflict, write a row in `${WORKING_FOLDER}/crm/.conflicts-<date>.md`:

```markdown
## <slug> — <table>

- field: <field>
  - markdown: <value> (modified <when>)
  - airtable: <value> (modified <when>)

resolve via: edit the markdown value and re-run sync, or change Airtable and re-run.
```

Never auto-resolve.

### 5. Update sync manifest

Rewrite `.airtable-sync.json`.

### 6. Print summary

```
direction: <push|pull|two-way>
pushed: <n>  pulled: <n>  unchanged: <n>  conflicts: <n>
manifest: .airtable-sync.json
conflicts file (if any): crm/.conflicts-<date>.md
```

## Guardrails

- Markdown is canonical. If in doubt, prefer markdown.
- A conflict is never silently merged. Always surfaced.
- `--dry-run` is the right move on the first run after a long gap.

## Failure modes

- Missing Airtable MCP or PAT → bail with setup steps.
- Schema mismatch (Airtable field renamed) → bail; print the mismatch and ask the user to fix the field name or update the mapping.
- Rate limit hit → checkpoint to manifest, exit cleanly, advise re-run.

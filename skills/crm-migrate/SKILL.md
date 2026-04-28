---
name: crm-migrate
description: One-shot migration between CRM backends. Markdown → Airtable, markdown → Attio, Airtable → markdown, Attio → markdown, or Airtable ↔ Attio. Use when first adopting an external CRM, when leaving one, or when swapping providers. Snapshot-style, not incremental.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(cp *), Bash(date *)
---

# CRM Migrate

The migration tool. The adapter skills (`crm-adapter-airtable`, `crm-adapter-attio`) are for ongoing two-way sync — this skill is the one-time copy. Always snapshots the source and writes a migration log.

## Inputs

`$ARGUMENTS`:
- **Required**: `--from=<markdown|airtable|attio>` and `--to=<markdown|airtable|attio>`.
- Optional: `--dry-run`.
- Optional: `--object=<companies|opportunities|outreach|all>` — default `all`.

### Examples

```
/career:crm-migrate --from=markdown --to=airtable
/career:crm-migrate --from=attio --to=markdown   # leaving Attio
/career:crm-migrate --from=airtable --to=attio   # swap providers
```

## Procedure

### 1. Pre-migration snapshot

Always copy the source state first.

- Source = markdown: tar `crm/` → `${WORKING_FOLDER}/cache/crm-snapshot-<date>.tar`.
- Source = airtable / attio: dump rows to `${WORKING_FOLDER}/cache/<source>-snapshot-<date>.json`.

### 2. Read source

- markdown: parse tables to row arrays.
- airtable / attio: pull all records via the relevant adapter's read path.

### 3. Map to destination shape

Use the field mappings from the relevant adapter SKILL.md. For unmappable fields (e.g. moving from Attio's free-form Notes to markdown row `notes` column), concatenate with `\n---\n` and put in a single column.

### 4. Confirm plan

Show:
- N companies, N opportunities, N outreach rows to migrate.
- Any fields that will be lossy (e.g. linked-record collapses, custom attrs not in destination).
- Destination already has K rows — will the migration *append* (skip slug collisions) or *overwrite*? Ask.

`--dry-run` stops here.

### 5. Apply

Write to destination. For external CRMs, prefer the adapter's create-by-slug path so re-running is idempotent.

For markdown destinations: write tables in the canonical layout. Preserve any existing rows the migration didn't touch (don't clobber a markdown CRM the user was already curating).

### 6. Migration log

Write `${WORKING_FOLDER}/crm/.migration-<date>.md`:

```markdown
# CRM migration — {{date}}

- from: {{source}}
- to: {{dest}}
- objects: {{which}}
- snapshot: cache/<...>
- migrated: {{counts}}
- skipped: {{counts + reasons}}
- lossy fields: {{list}}
```

### 7. Print summary

```
migrated: companies=<n> opportunities=<n> outreach=<n>
skipped: <n> (see log)
snapshot: cache/<source>-snapshot-<date>.<ext>
log: crm/.migration-<date>.md
```

## Guardrails

- Always snapshot before writing. Never destructive without a snapshot.
- Migrations are not incremental. After migration, switch to the relevant adapter for ongoing sync.
- If destination is non-empty, default to *append* (skip slug collisions) and prompt before *overwrite*.

## Failure modes

- Mid-migration error → snapshot exists, partial writes are tolerated (idempotent on re-run by slug).
- Source unreadable → bail before writing destination.

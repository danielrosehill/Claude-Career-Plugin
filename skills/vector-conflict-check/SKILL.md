---
name: vector-conflict-check
description: For CONTEXT_STORE=pinecone — verify the index against the local workspace. Flags vectors whose source file was deleted, renamed, or content-changed (hash mismatch), and surfaces vectors whose content contradicts current ground-truth (stale claims, closed engagements, outdated targets). Proposes deletes/re-embeds; never silently mutates the index.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(sha256sum *), Bash(test *), Bash(date *), Bash(mkdir *), mcp__jungle-personal__pinecone__describe-index-stats, mcp__jungle-personal__pinecone__search-records
---

# Vector Conflict Check

## When to run

- After bulk edits to ground-truth or CRM.
- Periodically (weekly) when running on Pinecone.
- Before relying on `semantic-recall` for high-stakes calls (offer comparisons, pivot decisions).

## Procedure

### 1. Resolve config + guard

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `CONTEXT_STORE`, `PINECONE_INDEX`. Bail if `CONTEXT_STORE != pinecone`.

### 2. Pull index inventory

Use the Pinecone MCP to enumerate vector IDs + metadata for the configured index. Each vector should carry metadata: `source_path`, `content_hash`, `embedded_at`, `namespace` (set by `pinecone-sync`).

If metadata is missing on a meaningful fraction of vectors, bail with: "index lacks source-path metadata — run `/career:pinecone-sync --reindex` first."

### 3. File-level checks

For each vector:

- **Source-deleted** → file path no longer exists in `WORKING_FOLDER`. Propose vector delete.
- **Source-renamed** → no exact-path match but a same-hash file exists elsewhere. Propose metadata update (cheap) over delete+re-embed (expensive).
- **Content-drift** → file exists at same path; sha256 mismatch with `content_hash` metadata. Propose re-embed (delete + upsert via `pinecone-sync`).
- **Clean** → path matches, hash matches. No action.

### 4. Semantic conflict checks

Read `${WORKING_FOLDER}/ground-truth.md` and extract assertion-shaped lines (looking-for, hard-constraints, location, currency, current-stage). For each assertion, search the vector index (`search-records`) with the assertion as query, top_k=10. For matches with high similarity (≥0.85) **but** semantic contradiction, flag.

Heuristic contradictions to detect (let the model judge, no regex needed):

- Vector says "looking for full-time" while ground-truth says "consulting only".
- Vector says "active engagement with X" while CRM marks X as `closed`.
- Vector contains a location/currency that no longer matches ground-truth.

### 5. Write findings

`${WORKING_FOLDER}/reconcile/<YYYY-MM-DD>-vector-conflicts.md`. Sections:

```
# Vector conflicts — {{date}}

## File-level
- DELETE  vec_id=...  source=research/companies/acme/...  reason=source-deleted
- REEMBED vec_id=...  source=ground-truth.md              reason=hash mismatch (last embedded 2026-01-12)
- RENAME  vec_id=...  source=domain-notes/foo.md → bar.md reason=path moved

## Semantic
- vec_id=...  excerpt="…still in talks with Acme…"  conflicts with: crm/companies.md:Acme=closed (2026-02-14)

## Proposed actions
- 12 deletes, 3 re-embeds, 1 rename, 4 semantic flags for user review
```

### 6. Apply (with explicit confirm)

`AskUserQuestion` per action class (delete batch / re-embed batch / rename batch / semantic flags). Semantic flags never auto-resolve — they're for the user to decide whether to update ground-truth, update the source doc, or keep both views.

Re-embeds delegate to `pinecone-sync --paths=<list>`.

## Idempotency

Re-running after a clean apply produces an empty findings file (or skips writing).

## Failure modes

- **Pinecone MCP missing** → bail with: "install via `/career:detect-mcps` or `/career:install-companion-plugins --only=private-misc`."
- **Index empty** → exit cleanly with "no vectors to check."
- **Network error mid-scan** → write partial findings, mark as `incomplete: true` in frontmatter.

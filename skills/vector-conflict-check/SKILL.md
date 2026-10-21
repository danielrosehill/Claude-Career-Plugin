---
name: vector-conflict-check
description: For CONTEXT_STORE=memory — verify the workspace namespace(s) on the gateway `memory` server (tools mcp__gateway__memory__*) against the local workspace. Flags memories whose source file was deleted, renamed, or content-changed (hash mismatch), and surfaces memories whose content contradicts current ground-truth (stale claims, closed engagements, outdated targets). Proposes deletes/re-stores; never silently mutates the store.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(sha256sum *), Bash(test *), Bash(date *), Bash(mkdir *), mcp__gateway__memory__list_namespaces, mcp__gateway__memory__list_memories, mcp__gateway__memory__recall, mcp__gateway__memory__update_memory, mcp__gateway__memory__forget
---

# Vector Conflict Check

## When to run

- After bulk edits to ground-truth or CRM.
- Periodically (weekly) when running on the `memory` backend.
- Before relying on `semantic-recall` for high-stakes calls (offer comparisons, pivot decisions).

## Procedure

### 1. Resolve config + guard

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `CONTEXT_STORE`, `MEMORY_NAMESPACE`, `vector_namespaces`. Bail if `CONTEXT_STORE != memory` (a legacy `pinecone` value counts as `memory`).

### 2. Pull namespace inventory

Page through `list_memories` (limit 100, increasing `offset`) for `MEMORY_NAMESPACE` and the applications namespace if configured. Each memory should carry the tags `memory-sync` writes: `path:` (or `source_path:`), `sha256:` (or `source_row_hash:`), plus its `source` line with the sync date. Cross-check against `.memory-sync.json`: an id in the namespace but not the manifest is an orphan.

If tags are missing on a meaningful fraction of memories, bail with: "namespace lacks source-path tags — run `/career:memory-sync --full` first."

### 3. File-level checks

For each memory:

- **Source-deleted** → file path no longer exists in `WORKING_FOLDER`. Propose `forget`.
- **Source-renamed** → no exact-path match but a same-hash file exists elsewhere. Propose a tag update via `update_memory` (cheap) over forget+re-store.
- **Content-drift** → file exists at same path; sha256 mismatch with `content_hash` metadata. Propose re-store (forget + remember via `memory-sync`).
- **Clean** → path matches, hash matches. No action.

### 4. Semantic conflict checks

Read `${WORKING_FOLDER}/ground-truth.md` and extract assertion-shaped lines (looking-for, hard-constraints, location, currency, current-stage). For each assertion, `recall` with the assertion as query, `namespaces` = the workspace namespace(s) only, limit=10. For matches with high similarity (≥0.85) **but** semantic contradiction, flag.

Heuristic contradictions to detect (let the model judge, no regex needed):

- Memory says "looking for full-time" while ground-truth says "consulting only".
- Memory says "active engagement with X" while CRM marks X as `closed`.
- Memory contains a location/currency that no longer matches ground-truth.

### 5. Write findings

`${WORKING_FOLDER}/reconcile/<YYYY-MM-DD>-vector-conflicts.md`. Sections:

```
# Vector conflicts — {{date}}

## File-level
- FORGET  mem_id=...  source=research/companies/acme/...  reason=source-deleted
- RESTORE mem_id=...  source=ground-truth.md              reason=hash mismatch (last embedded 2026-01-12)
- RENAME  mem_id=...  source=domain-notes/foo.md → bar.md reason=path moved

## Semantic
- mem_id=...  excerpt="…still in talks with Acme…"  conflicts with: crm/companies.md:Acme=closed (2026-02-14)

## Proposed actions
- 12 forgets, 3 re-stores, 1 rename, 4 semantic flags for user review
```

### 6. Apply (with explicit confirm)

`AskUserQuestion` per action class (forget batch / re-store batch / rename batch / semantic flags). Semantic flags never auto-resolve — they're for the user to decide whether to update ground-truth, update the source doc, or keep both views.

Re-stores delegate to `memory-sync --paths=<list>`.

## Idempotency

Re-running after a clean apply produces an empty findings file (or skips writing).

## Failure modes

- **`memory` MCP missing** (tools `mcp__gateway__memory__*` don't resolve via `ToolSearch`) → bail with: "no `memory` MCP available — run `/career:detect-mcps`."
- **Namespace empty** → exit cleanly with "no memories to check."
- **Network error mid-scan** → write partial findings, mark as `incomplete: true` in frontmatter.

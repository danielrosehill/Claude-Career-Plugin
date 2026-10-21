---
name: vector-init-applications
description: First-run scaffold of a memory namespace dedicated to application/outreach tracking. Creates a `<MEMORY_NAMESPACE>-applications` namespace on the gateway `memory` server (tools mcp__gateway__memory__*), sets the tag schema (company, role, status, applied_date, last_touch, source_path, stage), seeds it from crm/outreach.md + crm/opportunities.md, and registers it in config. Idempotent — re-running detects existing namespace and offers reseed or skip.
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash(date *), Bash(test *), mcp__gateway__memory__list_namespaces, mcp__gateway__memory__create_namespace, mcp__gateway__memory__list_memories, mcp__gateway__memory__remember, mcp__gateway__memory__forget, mcp__gateway__memory__recall
---

# Vector Init — Applications Namespace

A separate namespace for application tracking lets `semantic-recall` answer "have I already applied to a company like X?" or "what stage is the Acme conversation in?" without polluting the general workspace namespace.

## Procedure

### 1. Resolve config + guard

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `CONTEXT_STORE`, `MEMORY_NAMESPACE` (default `career-workspace`). Bail if `CONTEXT_STORE != memory` (point user at `/career:detect-mcps`). A legacy `CONTEXT_STORE=pinecone` is treated as `memory`.

Namespace name: `<MEMORY_NAMESPACE>-applications` (e.g. `career-workspace-applications`). `memory` namespaces are global to the user, so a bare `applications` would be ambiguous.

### 2. Check namespace state

`list_namespaces`. If the applications namespace exists with a non-zero count, ask: `reseed (clears + re-stores) | append (add new only) | skip`. If it doesn't exist, `create_namespace` with description "career-os application/outreach tracking, seeded by career:vector-init-applications".

### 3. Define tag schema

The `memory` server stores text plus `tags` and `source` (no free-form metadata, server-assigned ids), so structured fields travel as `key:value` tags. Each memory in the applications namespace carries:

```
{
  "namespace": "<MEMORY_NAMESPACE>-applications",
  "text": "<company> — <role> — <status> — <stage> — <last 1-2 message excerpts>",
  "tags": [
    "company:<slug>",
    "role:<title or 'consulting'>",
    "status:watching|applied|in-conversation|interview|offer|closed-won|closed-lost",
    "stage:<short stage note>",
    "applied_date:YYYY-MM-DD",          // omit if null
    "last_touch:YYYY-MM-DD",
    "source_path:crm/outreach.md|crm/opportunities.md|crm/companies.md",
    "source_row_hash:<first 12 hex of sha256 of the source row>"
  ],
  "source": "career:vector-init-applications <ISO timestamp>"
}
```

### 4. Seed from CRM

Read `${WORKING_FOLDER}/crm/outreach.md` and `${WORKING_FOLDER}/crm/opportunities.md`. Each row is a record. Build the embedding text by concatenating: company, role, status, stage, last 1-2 message excerpts. Hash the source row to populate `source_row_hash` (used later by `vector-conflict-check`).

`remember` one record per row (`allow_duplicate=true`; rows for the same company at different stages are legitimately similar). Record the returned ids in `.memory-sync.json` under the source file so `memory-sync` owns them. On reseed, page through `list_memories` for the namespace and `forget` every id first — there is no bulk delete.

### 5. Register in config

Update `${CAREER_DATA_DIR}/config.json`:

```json
{
  "vector_namespaces": {
    "default": "<MEMORY_NAMESPACE>",
    "applications": {
      "name": "<MEMORY_NAMESPACE>-applications",
      "seeded_at": "<ISO timestamp>",
      "sources": ["crm/outreach.md", "crm/opportunities.md"]
    }
  }
}
```

`memory-sync` reads `vector_namespaces.applications.sources` and routes those files to the dedicated namespace on subsequent syncs.

### 6. Verify

Run a sanity query: `recall` namespaces=[`<MEMORY_NAMESPACE>-applications`], query="recent outreach", limit=3. Show the user the matched rows so they can confirm the seed looks right.

### 7. Print next steps

```
Applications namespace seeded.
- Backend:   gateway memory server
- Namespace: <MEMORY_NAMESPACE>-applications
- Records:   <N>
- Tags:      company, role, status, stage, applied_date, last_touch, source_path

Next:
- /career:semantic-recall "<query>" --namespace=<MEMORY_NAMESPACE>-applications
- /career:vector-conflict-check                  — verify after CRM edits
- /career:memory-sync                            — incremental updates flow into the right namespace automatically
```

## Idempotency

- `skip` short-circuits.
- `append` only embeds rows whose `source_row_hash` is not yet in the namespace.
- `reseed` is destructive within the namespace only — never touches the default namespace.

## Failure modes

- **`memory` MCP missing** → same bail as `vector-conflict-check`.
- **`create_namespace` refused** (e.g. invalid name — lowercase letters, digits, dashes only) → report the error, suggest a valid name, mark config as `seeded_at: pending`.

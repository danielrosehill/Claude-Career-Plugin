---
name: vector-init-applications
description: First-run scaffold of a vector-store namespace dedicated to application/outreach tracking. Creates the `applications` namespace in the configured index, sets the metadata schema (company, role, status, applied_date, last_touch, source_path, stage), seeds it from crm/outreach.md + crm/opportunities.md, and registers it in config. Idempotent — re-running detects existing namespace and offers reseed or skip.
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash(date *), Bash(test *), mcp__gateway__pinecone__describe-index, mcp__gateway__pinecone__describe-index-stats, mcp__gateway__pinecone__upsert-records, mcp__gateway__pinecone__search-records
---

# Vector Init — Applications Namespace

A separate namespace for application tracking lets `semantic-recall` answer "have I already applied to a company like X?" or "what stage is the Acme conversation in?" without polluting the general workspace namespace.

## Procedure

### 1. Resolve config + guard

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `CONTEXT_STORE`, `PINECONE_INDEX`. Bail if `CONTEXT_STORE != pinecone` or `PINECONE_INDEX` unset (point user at `/career:detect-mcps`).

### 2. Check namespace state

`describe-index-stats` for the configured index. If namespace `applications` already exists with vectors, ask: `reseed (clears + re-embeds) | append (add new only) | skip`.

### 3. Define metadata schema

Each vector in `applications` carries:

```
{
  "id": "<source-path>:<row-hash>",
  "namespace": "applications",
  "metadata": {
    "company": "<slug>",
    "role": "<title or 'consulting'>",
    "status": "watching|applied|in-conversation|interview|offer|closed-won|closed-lost",
    "stage": "<freeform stage note>",
    "applied_date": "YYYY-MM-DD or null",
    "last_touch": "YYYY-MM-DD",
    "source_path": "crm/outreach.md|crm/opportunities.md|crm/companies.md",
    "source_row_hash": "<sha256 of the source row>",
    "embedded_at": "<ISO timestamp>"
  }
}
```

### 4. Seed from CRM

Read `${WORKING_FOLDER}/crm/outreach.md` and `${WORKING_FOLDER}/crm/opportunities.md`. Each row is a record. Build the embedding text by concatenating: company, role, status, stage, last 1-2 message excerpts. Hash the source row to populate `source_row_hash` (used later by `vector-conflict-check`).

Upsert in batches of ≤100. On reseed, delete the namespace first (`delete-by-namespace` if MCP supports; else delete-all-ids in batches).

### 5. Register in config

Update `${CAREER_DATA_DIR}/config.json`:

```json
{
  "vector_namespaces": {
    "default": "career",
    "applications": {
      "name": "applications",
      "seeded_at": "<ISO timestamp>",
      "sources": ["crm/outreach.md", "crm/opportunities.md"]
    }
  }
}
```

`pinecone-sync` reads `vector_namespaces.applications.sources` and routes those files to the dedicated namespace on subsequent syncs.

### 6. Verify

Run a sanity query: `search-records` namespace=`applications`, query="recent outreach", top_k=3. Show the user the matched rows so they can confirm the seed looks right.

### 7. Print next steps

```
Applications namespace seeded.
- Index:     <PINECONE_INDEX>
- Namespace: applications
- Records:   <N>
- Schema:    company, role, status, stage, applied_date, last_touch, source_path

Next:
- /career:semantic-recall "<query>" --namespace=applications
- /career:vector-conflict-check                  — verify after CRM edits
- /career:pinecone-sync                          — incremental updates flow into the right namespace automatically
```

## Idempotency

- `skip` short-circuits.
- `append` only embeds rows whose `source_row_hash` is not yet in the namespace.
- `reseed` is destructive within the namespace only — never touches the default namespace.

## Failure modes

- **Index doesn't exist** → bail with: "create the index first via `/career:detect-mcps` or `private-misc:personal-context-injection`."
- **Pinecone MCP missing** → same bail as `vector-conflict-check`.
- **Namespace creation unsupported by current MCP shape** → emit the equivalent JSON payload and instruct the user to run it manually; mark config as `seeded_at: pending`.

---
name: pinecone-sync
description: Incremental sync of working-folder/ into a Pinecone index for semantic recall. Embeds only changed files (mtime + content hash). Index name from config. Delegates first-time MCP setup to private-misc:personal-context-injection. Idempotent.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(find *), Bash(sha256sum *), Bash(stat *), Bash(mkdir *), Bash(date *)
---

# Pinecone Sync

Embeds the workspace into Pinecone so `semantic-recall` can pull in relevant context for downstream skills (recommendation, meeting-prep, ground-truth deltas). Skips files that haven't changed since the last sync.

## Inputs

`$ARGUMENTS`:
- Optional: `--full` — re-embed everything (rare; use after schema or model change).
- Optional: `--dry-run` — list what *would* be re-embedded, no writes.

## Prerequisites

- `private-misc` plugin installed (provides Pinecone MCP wiring). If missing, bail with: "install private-misc first via /career:onboard companions".
- Config keys: `PINECONE_INDEX`, `CONTEXT_STORE=pinecone`.

## Procedure

### 1. Load state

- `WORKING_FOLDER`.
- `${WORKING_FOLDER}/.pinecone-sync.json` — manifest of `{path, sha256, mtime, vector-id}`. Create if missing.
- `PINECONE_INDEX` from config.

### 2. Walk working-folder

Include:
- `ground-truth.md`
- `crm/*.md`
- `domain-notes/*.md`
- `recommendations/*.md`
- `research-briefs/*.md`
- `plans/*.md`
- `meetings/*.md`
- `inbox/*.md` (only `status: routed` entries — drafts churn too fast)

Exclude: `cache/`, `_salvage/`, `.pinecone-sync.json`, hidden files.

### 3. Diff against manifest

For each file: compute sha256. If hash differs from manifest (or file is new), mark for re-embed. If `--full`, mark all.

Mark deletions (in manifest, not on disk) for vector deletion.

### 4. Chunk + embed (proposed)

Chunking strategy:
- Markdown files: split on `## ` headers; keep heading + body as a single chunk; chunks > 2000 tokens further split on paragraph.
- One vector per chunk. Vector id: `<path>#<chunk-index>` (stable across runs).
- Metadata: `path, section, mtime, type` (where `type` ∈ ground-truth | crm | domain-note | recommendation | brief | plan | meeting | inbox).

If `--dry-run`: print the chunk plan and stop.

### 5. Upsert

Use Pinecone MCP `upsert-records` (namespace = workspace name) with the chunks. Delete vectors for files removed since last sync.

### 6. Update manifest

Rewrite `.pinecone-sync.json` with new `{path, sha256, mtime, vector-ids}` per file.

### 7. Print summary

```
synced: <N> files, <M> chunks
unchanged: <U> files
deleted: <D> vectors
index: <PINECONE_INDEX>
manifest: .pinecone-sync.json
```

## Guardrails

- Never embed `inbox/*` entries that are still `pending-route` — they're noisy and short-lived.
- `--full` re-embeds everything; warn the user about cost before running.
- Manifest is the source of truth for "what's already in the index"; don't query Pinecone to figure that out.

## Failure modes

- Pinecone MCP not installed → bail with install instructions.
- Index doesn't exist → offer to create via MCP `create-index-for-model`, then proceed.
- File unreadable → log and skip; don't abort the whole run.

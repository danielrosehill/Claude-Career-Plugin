---
name: memory-sync
description: Incremental sync of working-folder/ into a namespace of the `memory` upstream on the MCP gateway (tools mcp__gateway__memory__*) for semantic recall. Stores only changed files (content hash), forgets chunks of changed or deleted files. Namespace name from config. Idempotent.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(find *), Bash(sha256sum *), Bash(stat *), Bash(mkdir *), Bash(date *), mcp__gateway__memory__list_namespaces, mcp__gateway__memory__create_namespace, mcp__gateway__memory__remember, mcp__gateway__memory__forget, mcp__gateway__memory__list_memories
---

# Memory Sync

Stores the workspace in the gateway's `memory` server so `semantic-recall` can pull in relevant context for downstream skills (recommendation, meeting-prep, ground-truth deltas). Skips files that haven't changed since the last sync. Embedding happens server-side; nothing is embedded client-side.

## Inputs

`$ARGUMENTS`:
- Optional: `--full` — re-store everything (rare; use after a chunking change).
- Optional: `--dry-run` — list what *would* be re-stored, no writes.
- Optional: `--paths=<list>` — restrict the run to these files (used by `vector-conflict-check`).

## Prerequisites

- The `memory` upstream on the MCP gateway, i.e. tools `mcp__gateway__memory__*`. If they are not in the tool list, load them with `ToolSearch` (`select:mcp__gateway__memory__list_namespaces,mcp__gateway__memory__remember,mcp__gateway__memory__forget,mcp__gateway__memory__list_memories`). If they still don't resolve, bail with: "no `memory` MCP available — run `/career:detect-mcps`."
- Config keys: `CONTEXT_STORE=memory`, `MEMORY_NAMESPACE` (default `career-workspace`). A legacy config with `CONTEXT_STORE=pinecone` is treated as `memory`; tell the user to update the key.

## Procedure

### 1. Load state

- `WORKING_FOLDER`.
- `${WORKING_FOLDER}/.memory-sync.json` — manifest of `{path, sha256, mtime, memory_ids[]}`. Create if missing. (Older workspaces may have `.pinecone-sync.json`; ignore it — its vector ids don't exist in `memory`. Run with `--full` once.)
- `MEMORY_NAMESPACE` from config.
- `list_namespaces`. If the namespace doesn't exist, create it with `create_namespace` (description: "career-os workspace chunks synced by career:memory-sync"). Never write workspace chunks into a namespace that holds hand-written personal memories.

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

Exclude: `cache/`, `_salvage/`, `.memory-sync.json`, hidden files.

Files listed in `vector_namespaces.applications.sources` (see `vector-init-applications`) go to that namespace instead of `MEMORY_NAMESPACE`.

### 3. Diff against manifest

For each file: compute sha256. If hash differs from manifest (or file is new), mark for re-store. If `--full`, mark all.

Mark deletions (in manifest, not on disk) for `forget`.

### 4. Chunk (proposed)

Chunking strategy:
- Markdown files: split on `## ` headers; keep heading + body as a single chunk; chunks > 2000 tokens further split on paragraph.
- One memory per chunk. Prefix the text with `[<path> § <section>]` so a recalled chunk is self-describing.
- Tags (the `memory` server has no free-form metadata; tags carry it): `path:<path>`, `section:<heading>`, `type:<type>`, `sha256:<first 12 hex of file hash>`, `chunk:<index>` — where `type` ∈ ground-truth | crm | domain-note | recommendation | brief | plan | meeting | inbox.
- `source`: `career:memory-sync <YYYY-MM-DD>`.

If `--dry-run`: print the chunk plan and stop.

### 5. Write

For each changed file: `forget` every id in its manifest `memory_ids`, then `remember` each chunk (`namespace` = target namespace, `allow_duplicate=true` so near-identical boilerplate sections in different files are not silently merged). Record the returned ids. For deleted files: `forget` their ids.

### 6. Update manifest

Rewrite `.memory-sync.json` with new `{path, sha256, mtime, memory_ids}` per file.

### 7. Print summary

```
synced: <N> files, <M> chunks
unchanged: <U> files
forgotten: <D> memories
namespace: <MEMORY_NAMESPACE>
manifest: .memory-sync.json
```

## Guardrails

- Never store `inbox/*` entries that are still `pending-route` — they're noisy and short-lived.
- `--full` re-stores everything; warn the user it rewrites the whole namespace before running.
- Manifest is the source of truth for "what's already in the namespace"; don't page through `list_memories` to figure that out (that's `vector-conflict-check`'s job).
- Only ever `forget` ids that this manifest owns.

## Failure modes

- `memory` tools unavailable → bail with the prerequisite message.
- File unreadable → log and skip; don't abort the whole run.
- A `remember` fails mid-file → keep the ids that were written in the manifest for that file with `sha256: null`, so the next run retries it.

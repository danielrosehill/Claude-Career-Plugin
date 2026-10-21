---
name: semantic-recall
description: Query the `memory` upstream on the MCP gateway (tools mcp__gateway__memory__*), or fall back to grep-over-markdown, for context relevant to a task. Returns ranked snippets with cited paths. Used by suggest-companies, meeting-prep, weekly-plan, self-healing-context when CONTEXT_STORE=memory. Caller decides what to do with results.
disable-model-invocation: false
allowed-tools: Read, Bash(grep *), Bash(rg *), mcp__gateway__memory__recall
---

# Semantic Recall

Pulls relevant context from the workspace for the current task. Two backends:
- `CONTEXT_STORE=memory` → semantic search via the gateway `memory` server (`mcp__gateway__memory__recall`) over the namespace(s) `memory-sync` writes to. A legacy `CONTEXT_STORE=pinecone` is treated as `memory`.
- `CONTEXT_STORE=markdown` (default) → ripgrep over working-folder.

The skill returns *citations* — paths + chunks. The caller is responsible for using them.

## Inputs

`$ARGUMENTS`:
- **Required**: `--query="<text>"` — natural-language query.
- Optional: `--type=<ground-truth|crm|domain-note|recommendation|brief|plan|meeting|inbox|all>` — filter by chunk type. Default: `all`.
- Optional: `--top=<n>` — top-N results. Default: 8.
- Optional: `--min-score=<float>` — min relevance score (memory backend only). Default: 0.5.
- Optional: `--namespace=<name>` — query a different namespace (e.g. the applications namespace from `vector-init-applications`). Default: `MEMORY_NAMESPACE`.

### Examples

```
/* invoked internally by other skills, not via slash command */
semantic-recall --query="prior outreach to AI policy companies in Tel Aviv" --type=crm --top=10
semantic-recall --query="user's stated salary floor and reasoning" --type=ground-truth
```

## Procedure

### 1. Load config

- `CONTEXT_STORE`. If `memory`, also `MEMORY_NAMESPACE` (default `career-workspace`).

### 2a. Memory backend

- If the `mcp__gateway__memory__*` tools aren't loaded, load `recall` via `ToolSearch` (`select:mcp__gateway__memory__recall`).
- Call `recall` with `query`, `namespaces=[<namespace>]` (always pass it — never search across all namespaces, which would mix in unrelated personal memories), `limit` = `--top` (over-fetch ×2 when `--type` is set), `min_score` = `--min-score`.
- If `--type` is set, keep only results tagged `type:<type>`, then trim to `--top`.
- For each result, return: `{path, section, score, snippet, type}` — `path` / `section` / `type` come from the `path:` / `section:` / `type:` tags `memory-sync` writes; the snippet is the memory text.

### 2b. Markdown backend (fallback)

- `rg -n` over `working-folder/` matching files for `--type` (e.g. `--type=crm` → `crm/*.md`).
- Score by term-frequency match on query keywords (cheap heuristic).
- Return same shape minus `score` (use match-count instead).

### 3. Return

Print machine-parseable lines:

```
result: path=<path> section="<heading>" score=<float> type=<type>
snippet: <one-line preview>
```

…or a JSON block if caller passed `--json`.

## Guardrails

- Always cite the path. A recall result without a path is unusable downstream.
- Don't summarise the snippet — return raw text so the caller can decide.
- If `CONTEXT_STORE=memory` but `.memory-sync.json` is older than 24 hours, warn that recall may be stale.

## Failure modes

- `memory` MCP unavailable (tools don't resolve, or the call errors) → automatically fall back to markdown backend with a one-line note.
- Empty workspace → return zero results, not an error.

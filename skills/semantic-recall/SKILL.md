---
name: semantic-recall
description: Query Pinecone (or fallback to grep-over-markdown) for context relevant to a task. Returns ranked snippets with cited paths. Used by suggest-companies, meeting-prep, weekly-plan, self-healing-context when CONTEXT_STORE=pinecone. Caller decides what to do with results.
disable-model-invocation: false
allowed-tools: Read, Bash(grep *), Bash(rg *)
---

# Semantic Recall

Pulls relevant context from the workspace for the current task. Two backends:
- `CONTEXT_STORE=pinecone` → Pinecone semantic search.
- `CONTEXT_STORE=markdown` (default) → ripgrep over working-folder.

The skill returns *citations* — paths + chunks. The caller is responsible for using them.

## Inputs

`$ARGUMENTS`:
- **Required**: `--query="<text>"` — natural-language query.
- Optional: `--type=<ground-truth|crm|domain-note|recommendation|brief|plan|meeting|inbox|all>` — filter by chunk type. Default: `all`.
- Optional: `--top=<n>` — top-N results. Default: 8.
- Optional: `--min-score=<float>` — min relevance score (Pinecone only). Default: 0.6.

### Examples

```
/* invoked internally by other skills, not via slash command */
semantic-recall --query="prior outreach to AI policy companies in Tel Aviv" --type=crm --top=10
semantic-recall --query="user's stated salary floor and reasoning" --type=ground-truth
```

## Procedure

### 1. Load config

- `CONTEXT_STORE`. If `pinecone`, also `PINECONE_INDEX`.

### 2a. Pinecone backend

- Use Pinecone MCP `search-records`: query text, filter by `type` if provided, top-K = `--top`.
- Filter results below `--min-score`.
- For each result, return: `{path, section, score, snippet (from metadata), type}`.

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
- If `CONTEXT_STORE=pinecone` but the manifest is older than 24 hours, warn that recall may be stale.

## Failure modes

- Pinecone MCP unavailable → automatically fall back to markdown backend with a one-line note.
- Empty workspace → return zero results, not an error.

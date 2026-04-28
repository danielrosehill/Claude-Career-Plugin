---
name: research-router
description: Cost-routing helper. Reads RESEARCH_MODEL from config and decides whether a bulk-research call should run on Claude or be delegated to a cheaper model (DeepSeek Pro/Flash via OpenRouter). Reserves Claude for ground-truth, meeting-prep, draft-outreach, recommendation engine. Skills that produce many cheap research artifacts (discovery, ecosystem, financials briefs) call this first.
disable-model-invocation: false
allowed-tools: Read, Bash(test *)
---

# Research Router

Most career-os work is reasoning over user-specific context — Claude is right. But bulk web-shaped research (domain enumeration, ecosystem mapping, public-financials briefs) is cost-dominated. This skill is the routing decision: should *this* call use Claude, or hand off to a cheaper model.

The skill itself doesn't call any model — it returns a routing decision the caller acts on.

## Inputs

`$ARGUMENTS`:
- **Required**: `--task=<task-name>` — the calling skill or task type (e.g. `discover-by-domain`, `brief-company-financials`, `meeting-prep`).
- Optional: `--override=<model>` — force a specific routing target.

## Procedure

### 1. Load config

Read `config.yaml` (or workspace config). Keys:
- `RESEARCH_MODEL` — `claude` (default) | `deepseek-pro` | `deepseek-flash` | `auto`.
- `OPENROUTER_MCP` — name of installed OpenRouter MCP, if any.

If config missing, default to `claude` and note "no router configured."

### 2. Classify task

Two sets:

**Always Claude** (high-context, user-specific reasoning):
- `ground-truth`, `meeting-prep`, `draft-outreach`, `suggest-companies`, `route-capture`, `weekly-plan`, `weekly-log`, `self-healing-context`.

**Routable** (bulk, web-shaped, structurally similar across runs):
- `discover-by-domain`, `discover-by-geography`, `discover-remote-friendly`, `companies-like-x`, `chunked-discovery`.
- `brief-company-financials`, `brief-recruitment-profile`, `brief-glassdoor-signal`.
- `find-conferences`, `find-communities`, `find-jobs-boards`, `find-hackathons`, `map-investors`.
- `salary-research`, `branding-assessment`.

### 3. Decide

| RESEARCH_MODEL | Always-Claude task | Routable task |
| --- | --- | --- |
| `claude` | claude | claude |
| `deepseek-pro` | claude | deepseek-pro |
| `deepseek-flash` | claude | deepseek-flash |
| `auto` | claude | deepseek-flash if `OPENROUTER_MCP` set, else claude |

`--override` wins over the table.

### 4. Return routing decision

Print one line:

```
route: <claude|deepseek-pro|deepseek-flash> via <claude|openrouter:<mcp>>
reason: task=<task> RESEARCH_MODEL=<value>
```

Caller reads stdout and dispatches accordingly.

## Guardrails

- Never silently route an "Always Claude" task — those are quality-load-bearing.
- If `RESEARCH_MODEL=deepseek-*` but `OPENROUTER_MCP` not configured, downgrade to `claude` and warn.
- Routing is per-call; not sticky across a session.

## Failure modes

- Config unreadable → default to claude.
- Unknown task name → return `route: claude` with `reason: unknown task, defaulting to claude`.

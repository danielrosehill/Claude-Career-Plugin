---
name: detect-mcps
description: Scan the user's existing Claude Code plugins and MCP servers to detect capabilities the career plugin needs (email send, transcription, calendar/tasks, semantic store, CRM). For each capability, decide whether to reuse an already-installed plugin/MCP, install a recommended companion plugin, or add an MCP server. Writes the resolution map to ${CAREER_DATA_DIR}/config.json under a `capabilities` block. Invoked from /career:onboard before install-companion-plugins so we don't double up on capabilities the user already has.
disable-model-invocation: false
allowed-tools: Bash(claude plugin *), Bash(claude mcp *), Read, Write, Edit
---

# Detect MCPs

Map the capabilities the career plugin needs to whatever the user already has. Avoid forcing a companion plugin install when an existing MCP already covers the capability.

## Capabilities

| Capability | What career-os does with it | Default companion | Acceptable existing MCPs |
| --- | --- | --- | --- |
| `email-send` | `/career:outreach send` | `email-skills` | any MCP exposing a `send-email` / `send_email` tool (Gmail, Resend, Postmark, generic SMTP, gws-personal, gws-business) |
| `transcription` | `/career:capture` voice ingest | `claude-transcription` | any MCP exposing `transcribe_audio` / `transcribe-*` (gemini-transcription, AssemblyAI MCP, OpenAI-whisper MCP) |
| `calendar-and-tasks` | `/career:plan week`, `/career:meeting-prep`, route-capture | `schedule-manager` | any MCP exposing `create_event` AND `create_task` (gws + Todoist, Google Calendar MCP + a task system) |
| `semantic-store` | `/career:semantic-recall`, `pinecone-sync` | `private-misc` (sets up Pinecone) | any MCP exposing `upsert-records` + `search-records` (pinecone, weaviate, qdrant, ragie) |
| `social-sentiment` | `brief-glassdoor-signal`, `brief-cultural-fit`, `brief-recruitment-profile` | `social-feedback` | no equivalent MCP — companion plugin only |
| `decision-frameworks` | `compare-offer`, `meeting-prep` | `decision-evaluation-framework` | no equivalent MCP — companion plugin only |
| `crm` | outreach log, opportunities | built-in markdown | optional: Airtable MCP, Attio MCP for `CRM_ADAPTER` |
| `contact-discovery` | `find-contact`, `find-email-by-name`, `verify-email`, `enrich-contact`, `bulk-verify-crm` | bundled `hunter` MCP (this plugin's `.mcp.json`) | any MCP exposing `Domain-Search` + `Email-Finder` + `Email-Verifier` (Hunter, Apollo, etc.) |

## When to invoke

- During `/career:onboard`, immediately before `install-companion-plugins`. The detection map narrows what the install step actually offers.
- Standalone via `/career:detect-mcps` when the user changes their MCP setup and wants the resolution map refreshed.
- From any skill that hits a missing-capability error and wants to re-scan before bailing.

## Procedure

### 1. Resolve `CAREER_DATA_DIR` + load config

```bash
CAREER_DATA_DIR="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/career"
```

Read `${CAREER_DATA_DIR}/config.json` if it exists. If not, run with empty config — the resolution map will be written, the rest of onboarding will fill in `WORKING_FOLDER` and friends.

### 2. Enumerate installed plugins

```bash
claude plugin list --json 2>/dev/null || claude plugin list
```

Parse plugin names. Cross-check against the companion column of the table above. Mark each companion `installed` or `not-installed`.

### 3. Enumerate installed MCP servers + their tools

```bash
claude mcp list --json 2>/dev/null || claude mcp list
```

For each MCP server, capture: alias, scope (user / project / system), tool names. If `--json` isn't available, fall back to parsing `~/.claude.json` and any project-level `.mcp.json`.

If the system reminder at session start already lists deferred MCP tools (the long `mcp__<server>__<tool>` enumeration), prefer that — it's the live truth and skips a shell-out.

**Aggregator preference.** If `${CAREER_DATA_DIR}/config.json` has an `MCP_AGGREGATOR.alias`, treat tools served under that alias (e.g. `mcp__jungle-personal__*`) as the highest-priority candidates when multiple MCPs match a capability. Rationale: the user has already standardised on the aggregator and adding a separate MCP would duplicate.

**Inventory write-out.** Always write the full discovered tool list to `${CAREER_DATA_DIR}/mcp-inventory.json`:

```json
{
  "captured_at": "2026-04-30",
  "aggregator": "jungle-personal",
  "servers": [
    { "alias": "jungle-personal", "tools": ["mcp__gateway__hunter__Domain-Search", "..."] },
    { "alias": "todoist", "tools": ["create_task", "..."] }
  ]
}
```

Other career-os skills can read this file when they need to know whether a specific MCP tool is reachable without re-running discovery.

### 4. Match tools to capabilities

For each capability in the table, walk the tool list and check for a matching tool name. Examples:

- `email-send` → any tool whose name contains `send_email` or `send-email`. Record the matching MCP alias as a candidate.
- `transcription` → any tool matching `transcribe_audio*` or `transcribe-*`.
- `calendar-and-tasks` → require BOTH a `create_event` tool AND a `create_task` (or `create_todoist_task`) tool, possibly across different MCPs.
- `semantic-store` → any pair of `upsert-records` + `search-records` (or equivalent vector-db verbs).
- `contact-discovery` → tools matching `Domain-Search` AND `Email-Finder` AND `Email-Verifier` on the same alias. Hunter is the default; the plugin's bundled `.mcp.json` declares a `hunter` server, so this capability resolves to `existing-mcp:hunter` after the user enables it (env vars: `HUNTER_API_KEY`, optional `HUNTER_MCP_PACKAGE`).

If multiple candidates match (e.g. user has both `gws-personal` and `gws-business` for email), keep all of them and let the user choose during step 6.

### 5. Decide a resolution per capability

For each capability, the resolution is one of:

- `existing-mcp:<alias>` — user already has it; we'll wire career skills to call those MCP tools.
- `companion-plugin:<plugin>` — companion is installed (or will be installed); career skills delegate to its slash commands.
- `add-mcp:<recommended>` — user has neither; we propose adding a specific MCP server.
- `none` — unsupported; the dependent skills will warn at runtime.

Decision rules:

1. **Companion plugin already installed** → prefer it (consistent contract).
2. **Else existing MCP found** → use it. Record the exact tool names in the resolution.
3. **Else** → propose `companion-plugin:<default>`. Mention an `add-mcp` alternative if the user wants the lighter-weight path (e.g. add a Resend MCP instead of installing `email-skills`).
4. **CRM**: don't propose anything unless the user has chosen `airtable` or `attio` for `CRM_ADAPTER`.

### 6. Confirm with the user

Show a per-capability table:

```
capability           | candidate                          | action
email-send           | gws-personal (send_email)          | reuse existing MCP? [Y/n]
transcription        | none                               | install claude-transcription? [Y/n]
calendar-and-tasks   | gws-personal + Todoist MCP         | reuse both? [Y/n]
semantic-store       | pinecone (search-records, upsert)  | reuse existing MCP? [Y/n]
social-sentiment     | none                               | install social-feedback? [Y/n]
decision-frameworks  | decision-evaluation-framework      | already installed
```

For each row, use `AskUserQuestion`. The user may pick a different candidate when multiple matched.

### 7. Write the resolution to config

Update `${CAREER_DATA_DIR}/config.json` with a `capabilities` block:

```json
{
  "capabilities": {
    "email-send":          { "via": "existing-mcp", "alias": "gws-personal", "tool": "send_email" },
    "transcription":       { "via": "companion-plugin", "plugin": "claude-transcription" },
    "calendar-and-tasks":  { "via": "existing-mcp", "calendar": { "alias": "gws-personal", "tool": "create_event" }, "tasks": { "alias": "todoist", "tool": "create_task" } },
    "semantic-store":      { "via": "existing-mcp", "alias": "pinecone", "search-tool": "search-records", "upsert-tool": "upsert-records" },
    "social-sentiment":    { "via": "companion-plugin", "plugin": "social-feedback" },
    "decision-frameworks": { "via": "companion-plugin", "plugin": "decision-evaluation-framework" },
    "crm":                 { "via": "builtin-markdown" }
  }
}
```

Merge with any existing `capabilities` block — never blow away the user's prior choices silently.

Also keep the legacy top-level `companions` flags in sync (`companions.email`, `companions.transcription`, etc.) so older skills that read those don't break. New skills should prefer `capabilities`.

### 8. Hand off

- If any capability resolved to `companion-plugin:` and that plugin is `not-installed`, hand the install list to the `install-companion-plugins` skill via `--only=<plugin1,plugin2>`.
- If any resolved to `add-mcp:`, print a single `claude mcp add` command per server for the user to run (do not auto-execute — adding MCPs touches user-scope config).
- If everything is `existing-mcp:` or already-installed, print "no installs needed" and return.

## How other skills consume the map

Skills that touch a capability should read `capabilities.<capability>` from `${CAREER_DATA_DIR}/config.json` and dispatch:

- `via: existing-mcp` → call the named MCP tool directly (skill should accept a generic `email`, `transcribe`, `create-event` shim).
- `via: companion-plugin` → invoke the companion's slash command or skill.
- `via: add-mcp` (still pending) → bail with: "run `/career:detect-mcps` and accept the proposed MCP add."

This keeps career-os portable: a user with a heavy MCP setup won't be force-fed companion plugins; a user with nothing will get the default companion stack.

## Idempotency

- Re-running re-detects from scratch. Existing `capabilities` choices are shown as the current state and the user can keep or change each one.
- Removing an MCP between runs and re-detecting will surface the gap.

## Failure modes

- **`claude plugin list` / `claude mcp list` not available** — fall back to parsing `~/.claude.json` and any `.mcp.json` files in the workspace. Bail if neither yields data.
- **Multiple matching MCPs and the user picks none** — record `via: none` and warn that the dependent skills will fail until resolved.
- **Existing MCP tool name doesn't match any known shape** — list as a candidate but mark `confidence: low`; require explicit user confirmation.

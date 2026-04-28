---
name: import-linkedin-history
description: Parse a LinkedIn data export ZIP — positions, connections, messages, profile — and land structured markdown/CSV in <WORKING_FOLDER>/linkedin/. Surfaces follow-up candidates (recent connections + recent message threads) for the user to consider routing to outreach.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(unzip *), Bash(mkdir *), Bash(test *), Bash(ls *), Bash(file *), Bash(head *), Bash(wc *)
---

# Import LinkedIn History

Accept a LinkedIn data export ZIP, parse it, land structured outputs. Optionally cross-reference connections against the user's target lists.

## Inputs

`$ARGUMENTS`:
- **Required**: `<zip-path>` — path to the LinkedIn export ZIP (the "Get a copy of your data" download).
- Optional: `--cross-ref=<domain-slug>` — flag connections at companies in this domain note.
- Optional: `--surface-followups` — surface recent connections / message threads as outreach candidates.

### Examples

```
/career:ecosystem linkedin-import ~/Downloads/Basic_LinkedInDataExport_2026-04-28.zip
/career:ecosystem linkedin-import ~/Downloads/li-export.zip --cross-ref=ai-eval-startups --surface-followups
```

## Procedure

### 1. Validate + extract

Check ZIP exists. Extract to `${WORKING_FOLDER}/linkedin/raw/` (overwrite on re-run with confirmation).

LinkedIn exports typically contain: `Profile.csv`, `Positions.csv`, `Education.csv`, `Connections.csv`, `messages.csv`, `Endorsement_Received_Info.csv`, `Skills.csv`, `Recommendations_Received.csv` (exact filenames vary by export tier).

Inventory the files present and the row counts.

### 2. Parse + structure

Land parsed outputs:

- `${WORKING_FOLDER}/linkedin/profile.md` — single document with profile + positions + education + skills + endorsements summary.
- `${WORKING_FOLDER}/linkedin/connections.csv` — normalized columns: `name, headline, company, role, connected_on, profile_url`.
- `${WORKING_FOLDER}/linkedin/messages/<thread-id>.md` — one file per thread (or one consolidated `messages.md` for sparse exports).

### 3. Cross-reference (if `--cross-ref`)

For each connection, check whether their `company` matches a company in `domain-notes/${cross-ref}.md`. Tag matches in `${WORKING_FOLDER}/linkedin/cross-ref-${cross-ref}.md`:

```markdown
# LinkedIn × {{domain}} overlap

| connection | role | company (matched) | connected_on | last_message_at | suggested action |
```

### 4. Surface follow-ups (if `--surface-followups`)

- Recent connections (last 90 days) with no follow-up message → list as `## Cold connections — never followed up`.
- Active message threads with the user as last sender > 14 days ago → list as `## Stalled threads — your move`.
- Active threads with the counterparty as last sender → list as `## Threads waiting on you`.

Write to `${WORKING_FOLDER}/linkedin/follow-ups.md`. Don't auto-route to outreach.

### 5. Print summary

```
linkedin imported: linkedin/
profile + positions + connections + messages: parsed
connections: <n>
messages: <n> threads
cross-ref overlaps: <n> (if --cross-ref)
follow-up candidates: <n> (if --surface-followups)

next:
  cross-ref → /career:outreach find <company> (already-connected? skip Hunter)
  follow-ups → review linkedin/follow-ups.md, route to /career:outreach draft for the strongest 2–3
```

## Guardrails

- **Don't write back to LinkedIn.** This skill parses an export — it doesn't post, message, or connect.
- Connections data is private. Files land in `${WORKING_FOLDER}/linkedin/` which the workspace `.gitignore` should already cover via `linkedin/` (add it if not).
- When suggesting follow-ups, don't draft messages — that belongs to `draft-outreach`. This skill surfaces candidates only.

## Failure modes

- **ZIP doesn't look like a LinkedIn export** → bail with a list of files found vs. expected filenames.
- **Some columns missing** (LinkedIn export tier varies) → continue with the columns present; mark missing fields explicitly.
- **No connections file in export** → write profile-only output and warn.

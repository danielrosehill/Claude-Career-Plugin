---
name: find-jobs-boards
description: Enumerate specialist job boards relevant to a domain (and optionally the user's location/remote preference). Filters out generic boards (LinkedIn, Indeed) by default — surfaces only niche-aligned sources. Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>/jobs-boards.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Find Job Boards

Specialist boards, not generic. The user already knows about LinkedIn — surface the boards they wouldn't find on their own.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>`.
- Optional: `--location=<override>` (default `USER_LOCATION`).
- Optional: `--remote-only` — only boards that filter to remote.
- Optional: `--include-generic` — include LinkedIn / Indeed / Glassdoor.
- Optional: `--n=<count>` (default 12).

### Examples

```
/career:ecosystem boards "ai evals"
/career:ecosystem boards "devtools" --remote-only
/career:ecosystem boards "policy tech" --location="EMEA"
```

## Procedure

### 1. Paths

Output: `${WORKING_FOLDER}/domain-notes/${domain-slug}/jobs-boards.md`.

### 2. Enumerate

For each candidate board, capture:
- Name + URL.
- Niche fit (one line): why this board is on-topic.
- Posting volume signal (rough postings / month).
- Geography filter (does it scope to user's location).
- Cost (free for job-seekers).
- Notable employers visible on the board.
- Aggregator-vs-direct: does it list direct employer postings or recycled aggregator content.

### 3. Write the note

```markdown
# Job boards — {{domain}}

> Domain: {{domain}}
> Location: {{location}}
> Remote-only: {{true|false}}
> Surveyed: {{date}}

## Niche-aligned boards

| name | URL | volume | geo filter | direct/aggregated | notable employers | notes |

## Adjacent boards (broader scope, still useful)

(same columns)

## Generic boards

(only if --include-generic)

## Sources

1. ...
```

### 4. Print summary

```
note: domain-notes/<domain-slug>/jobs-boards.md
niche-aligned: <n>
adjacent: <n>

next: set up a saved-search alert on the top 2–3
```

## Guardrails

- "Direct vs aggregated" matters — aggregator-only boards have stale postings. Mark explicitly.
- Don't recommend boards that haven't been updated in the last 60 days — flag as `dormant` and surface separately.
- One row per board, even if it covers multiple domains.

## Failure modes

- **Niche has no specialist boards** → write empty niche-aligned section + populated adjacent section.
- **All boards require login to assess** → mark each `volume: gated` and continue.

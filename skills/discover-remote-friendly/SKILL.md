---
name: discover-remote-friendly
description: Enumerate companies in a domain that are genuinely remote-friendly to USER_LOCATION (not just "remote-OK" with US-only fine print). Writes to <WORKING_FOLDER>/domain-notes/remote-<domain-slug>.md. Honest negatives over hopeful positives.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Discover Remote-Friendly

Domain ∩ remote-friendly to `USER_LOCATION`. The bar: would *this user, in their actual timezone* clear the company's hiring filter today.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>` — the niche (free text).
- Optional: `--location=<override>` — defaults to `USER_LOCATION` from `~/.config/career-os/config.json`.
- Optional: `--n=<count>` (default 20).
- Optional: `--lens=<employer|client>` — defaults `employer`. `client` mode looks for companies that *buy* remote services rather than employ remotely.
- Optional: `--strict` — only "global remote" / "anywhere"; exclude "remote within EMEA" unless EMEA covers the user.
- Optional: `--refresh`.

### Examples

```
/career:discover remote-friendly "developer tools"
/career:discover remote-friendly "ai eval startups" --strict
/career:discover remote-friendly "policy research orgs" --lens=client
```

## Procedure

### 1. Resolve config + paths

Read `USER_LOCATION` and `WORKING_FOLDER` from config. Output: `${WORKING_FOLDER}/domain-notes/remote-${domain-slug}.md`.

### 2. Enumerate domain candidates

Same enumeration approach as `discover-by-domain`. Cap at ~2× target so the remote filter has headroom.

### 3. Test each candidate against USER_LOCATION

For each candidate, fetch:
- Live job postings (preferably engineering or comparable role) — look for explicit timezone language.
- Careers page "where we hire" / "remote policy".
- Public statements (LinkedIn, blog, YC profile).

Classify each as:
- `friendly` — explicit acceptance of the user's timezone/region in current postings.
- `friendly-with-overlap` — accepts globally but requires N hours overlap with US/EU. Note the overlap.
- `region-locked` — limited to a region that excludes the user.
- `us-only` — explicit US-only / US-payroll-only.
- `unclear` — no public signal.

`--strict` keeps only `friendly`.

### 4. Cross-reference outreach.md

Mark contact-status as in `discover-by-domain`.

### 5. Write the note

```markdown
# Remote-friendly companies in {{domain}} (for {{location}})

> Domain: {{domain}}
> Location filter: {{location}}
> Strict: {{true|false}}
> Surveyed: {{date}}

## Companies — friendly

| name | website | evidence (live posting / page) | overlap req | contact-status | notes |
| --- | --- | --- | --- | --- | --- |

## Companies — friendly with overlap

(same columns)

## Region-locked / US-only — for the record

| name | reason |

## Honest unknowns

| name | what would resolve this |

## Sources

1. ...
```

### 6. Print summary

```
note: domain-notes/remote-<domain-slug>.md
friendly: <n>
friendly-with-overlap: <n>
unclear: <n>
ruled-out: <n>

next: /career:suggest companies --from=remote-<domain-slug>
```

## Guardrails

- **Bias toward honest negatives over hopeful positives.** If a company says "remote-first" but every posting is "US only", classify as `us-only`, not `friendly`.
- Live postings beat marketing copy. If the careers page contradicts current job posts, trust the posts.
- Don't assume "remote" includes the user's timezone — verify or mark `unclear`.
- Mark inferred fields explicitly.

## Failure modes

- **No live postings + no public policy** → mark `unclear`, do not classify as `friendly`.
- **Postings contradict each other** → note the contradiction; classify by the most recent posting.

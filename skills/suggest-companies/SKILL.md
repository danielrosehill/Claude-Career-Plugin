---
name: suggest-companies
description: Rank companies from a domain note (or set of notes) for the user, applying anti-loop logic against outreach.md. Hard-excludes recently-contacted; rank-downs stalled; surfaces freshness so the logic is visible. Writes to <WORKING_FOLDER>/recommendations/companies-<date>.md.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Suggest Companies

The anti-loop spine. Reads a domain note (or several), reads `crm/outreach.md`, ranks candidates against `ground-truth.md`, and emits a ranked list with explicit `freshness` so the user can see *why* a company is or isn't on the list.

## Inputs

`$ARGUMENTS`:
- **Required (one of)**:
  - `--domain=<slug>` — single domain note.
  - `--from=<note-slug>` — any domain-notes file (e.g. `like-snowglobe`, `remote-devtools`, `geo-tel-aviv-ai`).
  - `--all-fresh` — every domain-notes file written in last 60 days, merged.
- Optional: `--n=<count>` (default 10).
- Optional: `--lens=<employer|client|partner>` (default `employer`).
- Optional: `--include-recent` — disable the hard exclusion (rare; for review of recent activity).
- Optional: `--explain` — verbose ranking rationale per row.

### Examples

```
/career:suggest companies --domain=agentic-policy-sim
/career:suggest companies --from=like-snowglobe --n=15 --explain
/career:suggest companies --all-fresh --lens=client
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`, `USER_LOCATION` from config. Output: `${WORKING_FOLDER}/recommendations/companies-$(date +%Y-%m-%d)-${source-slug}.md`.

### 2. Load inputs

- The selected domain note(s).
- `${WORKING_FOLDER}/ground-truth.md`.
- `${WORKING_FOLDER}/crm/outreach.md`.
- `${WORKING_FOLDER}/crm/companies.md` (for status overrides like `passed`).

### 3. Compute freshness for every candidate

For each company in the source note, derive `freshness`:

| value | rule |
| --- | --- |
| `never` | no row in outreach.md |
| `stale` | last outreach ≥ 90 days ago, status ∈ {sent, stalled, closed-lost} |
| `cooling` | last outreach 30–89 days ago, status ∈ {sent, stalled} |
| `recent` | last outreach < 30 days ago, OR active thread (replied/meeting-booked/meeting-done) within 90 days |
| `passed` | row in companies.md with status=passed (regardless of outreach) |
| `won` | status=closed-won within 12 months |

### 4. Apply anti-loop logic

Hard rules (cannot be overridden without `--include-recent`):
- Drop `recent`.
- Drop `passed` unless ground-truth.md was updated since the pass-date (re-evaluate signal).
- Drop `won` (already engaged).

Rank-down (still listed, marked):
- `cooling` → -3 ranking points.
- `stale` → -1 ranking point (slight; staleness ≠ disinterest).

### 5. Score remaining candidates

Score = sum of:
- **Domain fit** (0–5) — match between domain note's scope and ground-truth `domains-of-interest`.
- **Hard-constraints clear** (0/-∞) — if ground-truth `hard-constraints` are violated (e.g. US-only when user is in Israel), drop entirely.
- **Stage fit** (0–3) — match between company stage and ground-truth `looking-for`.
- **Lens fit** (0–3) — does the company shape (size, buying model) work for the chosen `--lens`.
- **Salary band signal** (-2..+2) — penalize when known band is far below ground-truth `salary` floor.
- **Anti-loop adjustment** (from step 4).

Top `--n` by score.

### 6. Write the recommendations file

```markdown
# Company suggestions — {{date}}

> Source: {{source-note(s)}}
> Lens: {{lens}}
> Anti-loop: {{enabled|disabled}}
> Generated: {{datetime}}

## Top picks

| rank | name | score | freshness | why | next-action |
| --- | --- | --- | --- | --- | --- |
| 1 | ... | 11 | never | one-line rationale | /career:research-company <slug> |

## Excluded (anti-loop)

| name | reason |
| --- | --- |
| Acme | recent — outreach 2026-04-21 |
| Globex | passed 2026-02-15 — ground-truth unchanged since |

## Rank-downs (still listed)

| name | freshness | adjustment | note |

## Why these and not others

- {{1–2 sentences linking ranking back to ground-truth}}

{{if --explain}}
## Per-row rationale

### {{name}}
- domain fit: 5 (matches "agentic AI for policy")
- stage fit: 3 (Series A; user wants pre-PMF or post-PMF early)
- lens fit: 2 (b2b saas, fits employer lens)
- salary signal: 0 (no public band)
- anti-loop: 0 (never)
- **score: 10**
{{/explain}}

## Sources

- {{domain-note path}}
- crm/outreach.md (read at {{datetime}})
- ground-truth.md (last modified {{date}})
```

### 7. Print summary

```
recommendations: recommendations/companies-<date>-<source>.md
candidates considered: <n>
excluded (anti-loop): <n>
rank-downs: <n>
top picks: <n>

next: /career:research-company <slug>  (for top pick)
```

## Guardrails

- **Run twice in succession → results must differ or explicitly explain why they're identical.** The freshness column makes this visible — if two consecutive runs produce the same top 10 with the same freshness, surface that as `## Stable list — no new signal since {{prev-run}}` rather than silently repeating.
- Anti-loop is the default. `--include-recent` is for review, not for "I want to contact them again anyway".
- Source-cite every freshness claim back to a specific outreach.md row (date + status).
- Don't invent companies. Only rank what's in the source note(s).

## Failure modes

- **No source note found** → bail with instruction to run a discovery skill first.
- **All candidates excluded by anti-loop** → write the file with empty top-picks but full excluded list, and recommend running a different discovery (new domain, or `--lens` change).
- **ground-truth.md missing required fields** → warn, score with what's present, mark scoring as `## Status: partial`.

## Idempotency

- Each run writes a new dated file — never overwrites. The recommendations directory becomes a log of how thinking evolved.

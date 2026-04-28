---
name: brief-company-overview
description: Produce a Company Overview research brief for a target company — name, locations, headcount, funding rounds at a glance, founder story, competitive landscape, vision, differentiators. Use when the user wants a top-level read on a potential employer or consulting client. Reads the template at templates/research-briefs/company-overview.md and writes to <WORKING_FOLDER>/research/companies/<slug>/company-overview.md. Cites at least 3 sources. Skipped fields are marked unknown, never fabricated.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Company Overview

Top-level company read. The first brief most users want. Other briefs (financials, recruitment, remote, glassdoor, cultural-fit) cover specific angles; this one is the executive summary.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>` — first positional.
- Optional: `--lens=<employer|client|partner>` (default `employer`).
- Optional: `--slug=<slug>` (default: kebab-case of company name).
- Optional: `--url=<homepage>` — short-circuits domain discovery.

## Procedure

### 1. Resolve config + paths

Read `${CAREER_DATA_DIR}/config.json` for `WORKING_FOLDER`. Compute `slug` (provided or derived). Output path: `${WORKING_FOLDER}/research/companies/${slug}/company-overview.md`.

If the file already exists, ask: overwrite, append, or open existing.

### 2. Read the template

Read `${PLUGIN_ROOT}/templates/research-briefs/company-overview.md`. The frontmatter declares required/optional inputs and minimum citations. Headings + guardrails are the contract; honour them.

### 3. Research

Surface order — cite at least 3 independent sources:

1. **Primary** — company homepage, careers, about, blog (counts as 1 source max).
2. **Secondary** — press coverage (TechCrunch, business press), Crunchbase / PitchBook public pages, Wikipedia (treat as pointer to primary sources).
3. **Tertiary** — recent CEO/CTO podcast appearances, conference talks, LinkedIn posts.

Use WebSearch for discovery, WebFetch for fetching. Prefer freshness: anything older than 18 months is stale for "recent moves."

### 4. Fill the template

Section by section. For each field:
- Sourced fact → fill with citation marker.
- Inferred from sourced facts → mark `(inferred)`.
- Cannot be sourced → write `unknown`. Never fabricate.

Numbers (headcount, funding) belong in this brief at low resolution (round to nearest hundred / round figure). Detailed numbers go in `company-financials`.

### 5. Write output

Replace the template's `{{placeholders}}` with filled values; keep the frontmatter structure (update `date`, `source_count`).

Write to the resolved output path. `mkdir -p` the parent dir.

### 6. Update CRM

Append a row to `${WORKING_FOLDER}/crm/companies.md` if the slug is new:

```
| <slug> | <name> | <domain> | <lens> | researched | <YYYY-MM-DD> | company-overview | |
```

If the row exists, update `last-touch` and append `+overview` to the `research-brief` column.

### 7. Print summary

```
company-overview written: research/companies/<slug>/company-overview.md
sources cited: <n>
unknowns: <list of fields left as unknown>
```

## Guardrails

- Min 3 independent sources. Company-controlled = 1 max.
- No marketing tagline as the description.
- Numbers stay rough here; precise figures live in `company-financials` brief.
- `unknown` is acceptable; fabrication is not.

## Failure modes

- **Config missing** → bail; instruct user to run `/career:onboard`.
- **Network failure on WebSearch** → write what you have so far with `unknown` for the rest; mark the file with `## Status: partial — missing sources for <fields>`.
- **Company name ambiguous** (multiple matches) → present top candidates and ask the user which one.

## Idempotency

- Re-run with same inputs → user is prompted before overwriting.
- Re-run with `--refresh` flag → silently overwrites.

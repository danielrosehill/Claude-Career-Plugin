---
name: brief-company-financials
description: Produce a Company Financials research brief — funding rounds, valuation, revenue (if disclosed), burn / runway, profitability signal, public-market metrics. Use when the user needs a numbers-only read for negotiation, due-diligence, or to gauge runway risk. Reads templates/research-briefs/company-financials.md and writes to <WORKING_FOLDER>/research/companies/<slug>/company-financials.md. Numbers are sourced or marked "not disclosed" — never estimated.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Company Financials

Numbers only. No narrative.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>`.
- Optional: `--slug=<slug>`.
- Optional: `--ticker=<symbol>` if public.

## Procedure

### 1. Resolve config + paths

Read `~/.config/career-os/config.json` → `WORKING_FOLDER`. Output: `${WORKING_FOLDER}/research/companies/${slug}/company-financials.md`. Confirm before overwrite.

### 2. Read template

`${PLUGIN_ROOT}/templates/research-briefs/company-financials.md`.

### 3. Source numbers

Priority order:

1. **SEC filings** (10-K, 10-Q, S-1, Form D) — public.
2. **Funding-round disclosures** — Crunchbase, PitchBook public pages, company press release.
3. **Reputable press** — Bloomberg, FT, WSJ, TechCrunch with sourced numbers.
4. **Levels.fyi / Glassdoor self-reports** — only for ranges in `Revenue (if disclosed)`, never as primary source.

Use WebSearch + WebFetch. Always capture the URL for each figure.

### 4. Fill the template

Every numeric field:
- Has an explicit currency.
- Has a source URL.
- Is dated (as-of).
- Is marked "company stated" / "press reported" / "SEC filing".

If undisclosed: write `not disclosed`. Do not estimate. Do not extrapolate from headcount or "industry averages."

### 5. Write output

Same as `brief-company-overview` step 5.

### 6. Update CRM

Append `+financials` to the company row's `research-brief` column.

## Guardrails

- **No invented numbers.** Period.
- Distinguish "company stated" vs "press reported" vs "filing" on every line.
- Currency explicit on every figure.
- Private companies with no disclosure → most fields are `not disclosed` and that's fine.
- For public companies, prefer the latest 10-K / 10-Q over secondary press.

## Failure modes

- Same as `brief-company-overview`.
- **Multiple shell entities under same brand** → ask the user which legal entity to research.

## Idempotency

- Re-run prompts before overwrite.
- `--refresh` silently overwrites.

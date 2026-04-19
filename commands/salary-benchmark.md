---
description: White-hat salary benchmarking for a role, company, or market — using only publicly available compensation data.
---

# /salary-benchmark

Research compensation ranges for a target role using exclusively public sources. Store findings in the workspace for later comparison.

## White-hat rule

All research is white-hat. Use only:

- Public job postings (employer sites, LinkedIn, Indeed, etc.)
- Published salary platforms (Levels.fyi, Glassdoor, Payscale, Salary.com, LinkedIn Salary Insights)
- Government wage data (BLS, Eurostat, national statistics offices)
- SEC / regulatory filings (exec comp)
- Anonymous community sources (Blind, Fishbowl) — treat as signal, not fact
- Published compensation surveys and vendor reports

Never impersonate recruiters, submit fake applications, scrape behind auth, or pressure contacts to leak bands.

## Steps

### 1. Target

Parse `$ARGUMENTS` (role title, company, or slug). Otherwise ask: role title, seniority, company (optional), location/market, currency.

### 2. Research

For the target:

- Pull published ranges from 3+ salary platforms where possible.
- Capture base / bonus / equity components separately where the source distinguishes them.
- Note per-company level equivalencies (e.g. Google L5 ≈ Meta E5 ≈ Amazon L6) where relevant.
- Apply geographic adjustments if the user's market differs from the source data.
- Flag confidence per data point: `Confirmed` (published by employer / regulator), `Reported` (salary platform aggregate), `Inferred` (triangulated).

### 3. Write findings

Save to `analysis/landscape/<role-slug>.md` (salary-research variant) or `outputs/salary/<role-slug>.md` (other variants). Use this shape:

```markdown
# <Role> — salary benchmark

**Date:** YYYY-MM-DD
**Market:** <geo / remote>
**Currency:** <CCY>

## Summary

<range p25 / p50 / p75 / p90 for total comp, one line>

## By company

### <Company>
- **Source:** <url>
- **Confidence:** Confirmed / Reported / Inferred
- | Level | Title | Base | Bonus | Equity (annual) | Total |
- **Notes:** signing, refreshers, vest schedule, etc.

## Cross-company table
...

## Level equivalency
...

## Geographic adjustment
...

## Gaps / what we couldn't find
...
```

### 4. Present

Summarise: median total comp, spread, which companies pay above market, and recommended target ask for the user's seniority and market.

---
name: salary-research
description: White-hat salary benchmarking for a (geography, role) pair. Cached at cache/salary-<geo>-<role>.md so repeat calls don't re-fetch. Public sources only — postings, Levels.fyi, Glassdoor, Payscale, BLS/Eurostat, SEC, Blind/Fishbowl as signal. Confidence-tagged per data point.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Salary Research

Compensation ranges for a target role in a given market, using only publicly available data. Cached so you don't waste a search budget on the same query.

## Inputs

`$ARGUMENTS`:
- **Required**: `--role=<title>` and `--geo=<market>` (e.g. `--role="staff-pm" --geo="tel-aviv"`, or `--geo=remote-emea`).
- Optional: `--seniority=<junior|mid|senior|staff|principal>` — disambiguates if role doesn't already imply.
- Optional: `--currency=<CCY>` — default = market default (USD for US/remote, EUR for EU, ILS for Israel).
- Optional: `--company=<slug>` — narrows to a specific employer.
- Optional: `--refresh` — ignore cache and re-fetch.

### Examples

```
/career:salary --role="staff-pm" --geo="tel-aviv"
/career:salary --role="ai-researcher" --geo=remote-emea --refresh
/career:salary --role="solutions-architect" --geo="dublin" --company=stripe
```

## White-hat rule

Use only:
- Public job postings (employer sites, LinkedIn, Indeed).
- Salary platforms (Levels.fyi, Glassdoor, Payscale, Salary.com, LinkedIn Salary Insights).
- Government wage data (BLS, Eurostat, national stats).
- SEC / regulatory filings (exec comp).
- Anonymous community sources (Blind, Fishbowl) — *signal*, not fact.
- Published comp surveys.

Never impersonate recruiters, submit fake applications, scrape behind auth, or pressure contacts to leak bands.

## Procedure

### 1. Resolve cache path

`${WORKING_FOLDER}/cache/salary-<geo-slug>-<role-slug>.md`. If exists and `--refresh` not set and mtime < 90 days, read it and short-circuit to step 5 (with a "cached" note).

### 2. (Optional) Routing decision

If config `RESEARCH_MODEL != claude`, call `research-router --task=salary-research` and dispatch the bulk fetch to the routed model.

### 3. Research

For the (role, geo) pair:
- 3+ salary platforms where possible.
- Capture base / bonus / equity separately when distinguishable.
- Per-company level equivalencies where relevant.
- Geographic adjustment if source data is in a different market.
- Confidence per data point: `Confirmed` (employer/regulator) | `Reported` (platform aggregate) | `Inferred` (triangulated).

If `--company` set, narrow to that employer plus 3-5 peers as comparison.

### 4. Write findings

Cache file shape:

```markdown
# {{role}} — {{geo}} salary benchmark

**Generated:** YYYY-MM-DD
**Currency:** {{CCY}}
**Sources used:** {{list}}

## Summary

p25 / p50 / p75 / p90 total comp: {{...}}

## By company

### {{company}}
- source: {{url}}
- confidence: Confirmed | Reported | Inferred
- | level | title | base | bonus | equity (annual) | total |
- notes: signing / refreshers / vest schedule.

## Caveats

- {{any market-specific notes, e.g. ILS volatility, EU equity rarity, etc.}}
```

### 5. Print summary

```
cache: cache/salary-<geo>-<role>.md  ({{cached|fresh}})
range: {{p25}}–{{p75}} ({{currency}})
sample: {{N companies, M data points}}
```

## Guardrails

- White-hat only. Don't roleplay as a recruiter or applicant for data.
- Tag confidence honestly. "Inferred" beats a fake "Confirmed."
- Cache hits don't get a refresh — explicitly require `--refresh`. The user is paying for the search budget.
- If geo is "remote", state which timezone bands the data covers; remote-PT and remote-EU pay differently.

## Failure modes

- Insufficient data (< 3 sources) → still write the cache, but mark `confidence: low` at the top and recommend `/career:research-company` for specific targets.
- Currency conversion ambiguous → use one clear CCY and note the exchange-rate date.

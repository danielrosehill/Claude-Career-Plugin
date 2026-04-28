---
description: Run a single research brief on a company. Use for ad-hoc deep-dives on one angle (e.g. just remote-friendliness, or just glassdoor signal) without paying the cost of a full six-brief run.
---

# /career:research-brief

Single-brief runner. Wraps the six brief skills directly.

## Arguments

`$ARGUMENTS`:
- **Required**: `<company_name>` — first positional.
- **Required**: `<brief>` — second positional. One of: `overview`, `financials`, `recruitment`, `remote`, `glassdoor`, `cultural`.
- Optional: `--lens=<employer|client|partner>` (default `employer`).
- Optional: `--slug=<slug>`.
- Optional: `--refresh` — overwrite existing file without prompting.
- Brief-specific options pass through (`--function=`, `--team=`, `--user-location=`).

### Examples

```
/career:research-brief snowglobe remote
/career:research-brief myrofish glassdoor --team=engineering
/career:research-brief acme cultural --lens=client
/career:research-brief acme overview --refresh
```

## Procedure

### 1. Validate args

If `<brief>` is missing or not in the allowed set, list the six options and ask.

### 2. Map to skill

| arg | skill |
| --- | --- |
| `overview` | `brief-company-overview` |
| `financials` | `brief-company-financials` |
| `recruitment` | `brief-recruitment-profile` |
| `remote` | `brief-remote-friendliness` |
| `glassdoor` | `brief-glassdoor-signal` |
| `cultural` | `brief-cultural-fit` |

### 3. Invoke

Pass `<company_name>` + lens / slug / refresh / brief-specific options through to the skill.

### 4. Print result

The skill emits its own output summary. No additional synthesis here — that's `/career:research-company` territory.

## When to use this vs `/career:research-company`

- **`/career:research-brief`** — you already have most briefs; you want one specific angle refreshed. Or you want to evaluate a single hypothesis (e.g. "is this place actually remote-friendly?") before committing time to the full run.
- **`/career:research-company`** — first-time deep-dive on a serious target. Six briefs + SUMMARY.

## Failure modes

- Inherit from the underlying skill.

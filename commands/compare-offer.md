---
description: Side-by-side comparison of job offers (or role opportunities) across comp, growth, and fit dimensions.
---

# /compare-offer

Generate a structured comparison between two or more offers/roles so the user can reason about them concretely.

## Steps

### 1. Identify what to compare

Parse `$ARGUMENTS` for slugs, company names, or role titles. If missing, list tracked roles from the workspace (`data/processes.json`, `analysis/roles/`, or `goals/roles/`) and ask the user which to pit against each other. Minimum two items.

### 2. Gather data for each item

For each offer/role, collect:

- **Comp:** base, bonus (% + structure), equity (RSU/option value, vest schedule, cliff), signing bonus, relocation.
- **Total Comp (Year 1 / Year 4):** computed.
- **Benefits:** health, retirement match, PTO, parental leave, study/CPD budget.
- **Growth:** promotion cadence, next-level expectation, scope.
- **Work:** hours expectation, remote policy, manager, team size.
- **Location:** cost-of-living adjustment needed, commute, tax implications.
- **Risk:** company stage, runway, layoff history, industry volatility.
- **Fit:** alignment with `context/profile.md` (if present) — stated goals, constraints, values.

If data is missing for a dimension, mark `unknown` rather than guessing.

### 3. Write the comparison

Save to `outputs/comparisons/<date>-<slug-a>-vs-<slug-b>.md` (create the folder if absent). Use this shape:

```markdown
# Offer comparison: <A> vs <B> [vs <C>]

**Generated:** YYYY-MM-DD

## At a glance

| Dimension | <A> | <B> |
|---|---|---|
| Base | | |
| Bonus | | |
| Equity (yr1 / yr4) | | |
| Total comp yr1 | | |
| Total comp yr4 | | |
| Benefits highlight | | |
| Growth | | |
| Work / location | | |
| Risk | | |
| Fit score (1-5) | | |

## Deep dive — <A>
...

## Deep dive — <B>
...

## Tradeoffs
- ...

## Open questions to resolve before deciding
- ...
```

### 4. Present

Summarise the top 3 tradeoffs and the top 3 unknowns the user should resolve before accepting/declining.

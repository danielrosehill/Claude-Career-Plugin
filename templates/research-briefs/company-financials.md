---
brief: company-financials
required-inputs: [company_name]
optional-inputs: [company_slug]
output-path: research/companies/{{slug}}/company-financials.md
min-citations: 3
---

# {{company_name}} — Financials

> Researched: {{date}}
> Sources cited: {{source_count}}

**Numbers only. No narrative interpretation. The cultural-fit and overview briefs handle qualitative reads.**

## Capitalisation

- **Total raised**: {{total_raised}} ({{currency}})
- **Latest valuation**: {{valuation}} ({{valuation_date}}, {{post_or_pre_money}})
- **Stage**: {{stage}}
- **Public ticker**: {{ticker_or_none}}

## Funding rounds

| Date | Round | Amount | Pre-money | Lead | Other participants | Source |
| --- | --- | --- | --- | --- | --- | --- |
| | | | | | | |

## Revenue (if disclosed)

| Period | Revenue | Source | Notes |
| --- | --- | --- | --- |
| | | | |

If private and undisclosed: write `not disclosed`. Do not estimate.

## Burn / runway (if disclosed)

| Period | Burn | Runway implied | Source |
| --- | --- | --- | --- |

If undisclosed: `not disclosed`.

## Profitability signal

- **Profitable?** {{yes | no | unknown}}
- **Statement source**: {{quote + url}}

## Public-market metrics (if applicable)

| Metric | Value | As-of | Source |
| --- | --- | --- | --- |
| Market cap | | | |
| TTM revenue | | | |
| TTM net income | | | |
| Cash on hand | | | |

## Sources

1. {{url}} — {{what was sourced}}
2. ...

## Guardrails

- Never invent numbers. If a field isn't sourced, mark `not disclosed` or `unknown`.
- Distinguish "company stated" vs "press reported" vs "filing".
- For private companies, prefer SEC Form D, Crunchbase, PitchBook, or PR-confirmed numbers over speculation.
- Currency must be explicit on every figure.

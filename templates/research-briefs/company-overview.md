---
brief: company-overview
required-inputs: [company_name]
optional-inputs: [lens, company_url, company_slug]
output-path: research/companies/{{slug}}/company-overview.md
min-citations: 3
---

# {{company_name}} — Company Overview

> Lens: {{lens|default:employer}}
> Researched: {{date}}
> Sources cited: {{source_count}}

## Snapshot

- **Legal name**: {{legal_name}}
- **Headquarters**: {{hq_location}}
- **Other offices**: {{other_offices}}
- **Founded**: {{founded_year}}
- **Headcount**: {{headcount}} (as of {{headcount_date}})
- **Stage**: {{stage}} (e.g. seed, Series B, public, bootstrapped)
- **Website**: {{website}}
- **Careers page**: {{careers_url}}

## What they do

{{2–4 sentences. Plain-English description of the product / service, who pays, what problem it solves. Avoid marketing copy.}}

## Founder story

{{Who founded it, when, why. Prior backgrounds. Notable founding context — pivot history if relevant.}}

## Vision / strategic direction

{{1–2 paragraphs. Where they're trying to go in the next 2–3 years. Cite recent CEO/CTO public statements, blog posts, podcast appearances.}}

## Differentiators

{{3–5 bullets. What makes them distinct from competitors. Concrete, not generic.}}

## Competitive landscape

| Competitor | Position vs {{company_name}} | Notes |
| --- | --- | --- |
| | | |

## Funding (high-level)

{{Brief — full numerics in company-financials brief.}}

| Round | Date | Amount | Lead |
| --- | --- | --- | --- |
| | | | |

## Recent moves (last 12 months)

- {{Hires, departures, product launches, M&A, fundraising, controversies, layoffs.}}

## Sources

1. {{url}} — {{what was sourced from it}}
2. {{url}} — ...
3. {{url}} — ...

## Guardrails (for the agent)

- Do not regurgitate the company's own marketing tagline as the description.
- Cite at least 3 independent sources (company-controlled = 1 max).
- Mark anything inferred (vs. directly cited) with `(inferred)`.
- If a field can't be sourced, write `unknown` — never fabricate.

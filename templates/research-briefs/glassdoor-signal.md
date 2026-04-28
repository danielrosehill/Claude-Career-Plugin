---
brief: glassdoor-signal
required-inputs: [company_name]
optional-inputs: [company_slug, target_team]
output-path: research/companies/{{slug}}/glassdoor-signal.md
min-citations: 3
---

# {{company_name}} — Glassdoor Signal

> Researched: {{date}}
> Target team: {{target_team|default:company-wide}}
> Sources cited: {{source_count}}

**Aggregate sentiment, not a recap of individual reviews. Anonymous reviews are signal, not fact.**

## Headline numbers

- **Overall rating**: {{rating}}/5 ({{n_reviews}} reviews)
- **CEO approval**: {{ceo_pct}}%
- **Recommend to friend**: {{recommend_pct}}%
- **Trend (last 12 months)**: {{up | flat | down}}, {{delta}} points
- **Volume of reviews (last 6 months)**: {{n}}

## Sub-ratings

| Dimension | Rating | Trend |
| --- | --- | --- |
| Career opportunities | | |
| Comp & benefits | | |
| Culture & values | | |
| Senior management | | |
| Work-life balance | | |
| Diversity & inclusion | | |

## Tenure data

- **Median tenure (overall)**: {{n}} years
- **Median tenure ({{target_team}})**: {{n}} years
- **Red-flag check**: {{if median tenure on the relevant team is < 1 year, flag here with quote+source. Don't flag for company-wide median in large orgs.}}

## Compensation data

Stratified by area where data is available:

| Role | Geography | Base | Bonus | Equity | Source | n |
| --- | --- | --- | --- | --- | --- | --- |
| | | | | | | |

## Theme analysis (top complaints)

{{The 3–5 themes that appear most often across negative reviews. Quote 1–2 representative reviewers per theme with date. Look for *patterns*, not outliers.}}

1. **{{theme}}** — {{n review mentions, mostly {{recent | historical}}.}}
   > "{{quote}}" — {{role, date}}

2. **{{theme}}** — ...

## Theme analysis (top positives)

{{Same pattern.}}

1. **{{theme}}** — {{n review mentions.}}
   > "{{quote}}" — {{role, date}}

## Recent direction

{{If reviews show a clear before/after — leadership change, layoffs, acquisition — call it out. Quote one review on each side of the inflection.}}

## Sources

1. Glassdoor — {{url}}
2. {{Levels.fyi / Blind / TeamBlind / Comparably / etc.}} — {{url}}
3. ...

## Guardrails

- **Don't over-index on outliers.** Single 1-star review with a personal grievance is not signal.
- **Don't over-index on age either.** A scathing review from 3 years ago about a CEO who left isn't relevant unless the same patterns recur.
- **Tenure red flags are scoped.** Sub-1-year median across a 5000-person company is meaningless. Sub-1-year median in a 30-person engineering team is loud.
- **Compensation data must cite source.** Levels.fyi self-reports are not equivalent to formal disclosure.
- **No invented quotes.** Every blockquote must be verbatim from a real review.

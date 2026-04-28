---
brief: remote-friendliness
required-inputs: [company_name, user_location]
optional-inputs: [company_slug]
output-path: research/companies/{{slug}}/remote-friendliness.md
min-citations: 3
---

# {{company_name}} — Remote Friendliness (honest appraisal)

> Researched: {{date}}
> User location: {{user_location}}
> Sources cited: {{source_count}}

**This brief gives the user an honest read on whether they would actually be eligible — not whether the company calls itself remote. Most "remote-first" companies have geographic restrictions that aren't visible until interview stage.**

## Verdict (top of file)

- **Eligible for {{user_location}}?** `yes | conditional | no | unclear`
- **Confidence**: `high | medium | low`
- **Headline reason**: {{one sentence}}

## Stated remote policy

{{Direct quote(s) from careers page / company blog / CEO statement. URL each one. No paraphrase.}}

## Geographic restrictions (real)

Pull from job postings, not marketing copy:

| Source | Posting | Location requirement | Time-zone requirement |
| --- | --- | --- | --- |
| | | | |

Typical patterns to flag:
- "Remote, US only"
- "Remote, EU only"
- "Remote within {{n}} hours of {{tz}}"
- "Hybrid 3 days in {{city}}"
- "Remote anywhere — but no contractor-only jurisdictions" (often excludes IL, IN, etc. for tax reasons)

## Marketing-vs-reality gap

{{If the careers page says "we are fully remote" but every recent job posting requires US W-2: call this out explicitly. Quote both. Cite both URLs.}}

## Time-zone reality

- **Engineering core hours**: {{e.g. 10am–2pm PT}}
- **Standup time**: {{if known}}
- **{{user_location}} delta**: {{n}} hours offset

## Hiring entity / contractor reality

- **Entities the company hires through**: {{e.g. Deel, Remote.com, direct W-2, EOR-only countries list}}
- **Direct hire in {{user_location}}?** `yes | no | unknown`
- **Contractor-friendly in {{user_location}}?** `yes | no | unknown`

## Existing employees in {{user_location}} or adjacent

{{Search LinkedIn for current employees in user's country or time zone. List ≤5 with role + tenure. Strong positive signal if present.}}

## Sources

1. {{url}} — {{what was sourced}}
2. ...

## Guardrails

- **Do not regurgitate the company's own remote messaging.** Treat marketing copy as one input, not the answer.
- Bias toward live job postings — they show what HR will actually approve.
- If you can't find a single live posting that matches the user's geography, the verdict is `no` regardless of what the careers page says.
- "Unclear" is an acceptable verdict — say so rather than guessing.
- This brief is the most consequential for IL-based / non-US-EU users. Optimise for honest negative answers over hopeful positive ones.

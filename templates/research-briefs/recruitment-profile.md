---
brief: recruitment-profile
required-inputs: [company_name]
optional-inputs: [company_slug, target_function]
output-path: research/companies/{{slug}}/recruitment-profile.md
min-citations: 3
---

# {{company_name}} — Recruitment Profile

> Researched: {{date}}
> Target function: {{target_function|default:any}}
> Sources cited: {{source_count}}

## Surfaces

- **Careers page**: {{careers_url}}
- **LinkedIn page**: {{linkedin_url}} ({{linkedin_followers}} followers)
- **Glassdoor page**: {{glassdoor_url}} (rating {{glassdoor_rating}}/5, {{glassdoor_review_count}} reviews)
- **Indeed page**: {{indeed_url}}
- **AngelList / Wellfound**: {{angel_url}}
- **Other**: {{other_recruiting_surfaces}}

## Open roles (current)

| Title | Function | Location | Remote? | Posted | Link |
| --- | --- | --- | --- | --- | --- |
| | | | | | |

Total open roles: {{open_role_count}}

## Open roles in target function ({{target_function}})

{{Subset of the above. Highlight if any.}}

## Headcount trend

| Date | LinkedIn employee count | Source |
| --- | --- | --- |
| | | |

Direction: {{growing | flat | shrinking}}. Magnitude: {{n}} over {{period}}.

## Hiring signals

- **Recent recruiter activity**: {{e.g. recruiters posting on LinkedIn this month}}
- **Recent senior hires**: {{name, role, prior — last 3 months}}
- **Recent senior departures**: {{name, role — if signal-bearing}}
- **Recent layoffs**: {{date, scope, public statement}}

## Hiring process notes

{{From Glassdoor "interview" reports. Typical stages, typical timeline, takehome / no-takehome, pay-band transparency. Cite reviewers' timestamps.}}

## Recruiter contacts (public)

| Name | Title | Channel | Source |
| --- | --- | --- | --- |
| | | | |

(Hunter MCP results from `find-contact` go here; this brief does not call Hunter directly.)

## Sources

1. {{url}} — {{what was sourced}}
2. ...

## Guardrails

- Distinguish "live posting" from "ghost role" — flag jobs posted >60 days that haven't moved.
- Don't cite Glassdoor anonymous reviews as fact; treat as signal.
- Headcount from LinkedIn is approximate; note the snapshot date.

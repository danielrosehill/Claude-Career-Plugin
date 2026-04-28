---
brief: cultural-fit
required-inputs: [company_name]
optional-inputs: [company_slug, lens]
output-path: research/companies/{{slug}}/cultural-fit.md
min-citations: 3
requires: [ground-truth.md]
---

# {{company_name}} — Cultural Fit

> Researched: {{date}}
> Lens: {{lens|default:employer}}
> Sources cited: {{source_count}}
> Ground truth read: {{ground_truth_last_modified}}

**Mixes external research with the user's own ground-truth document. The point is not "is this company nice" — it's "does this company's actual operating mode match the user's actual operating mode."**

## Verdict (top of file)

- **Cultural fit**: `strong | likely | uncertain | poor`
- **Confidence**: `high | medium | low`
- **Headline reason**: {{one sentence}}

## User's stated style (from ground-truth)

Pulled verbatim from `ground-truth.md`. If a constraint was missing there, ask the user inline.

- **Looking for**: {{employment | contract | both}}
- **Hard constraints (relevant subset)**: {{location, hours, family, other from ground-truth}}
- **Domains of interest**: {{from ground-truth}}
- **Collaboration preferences**: {{ask if not in ground-truth}}
- **Decision-making style**: {{ask if not in ground-truth}}
- **What drains you / what energises you**: {{ask if not in ground-truth}}

## Company's apparent operating mode

Pulled from public signals — engineering blog posts, podcast appearances, public Slack/Discord, careers-page values, Glassdoor culture themes, public memos.

- **Communication mode**: {{written-first | meeting-first | sync-heavy | async-first}}
- **Decision velocity**: {{startup-fast | deliberate | committee-heavy}}
- **Hierarchy**: {{flat | clear levels | shifting}}
- **Operating cadence**: {{quarterly OKRs | sprint | continuous | other}}
- **On-call / always-on expectations**: {{yes | no | role-dependent}}
- **Documented values**: {{list from careers page; flag if values feel performative vs operationalised}}
- **How feedback flows**: {{1:1-driven | written reviews | unclear}}

## Match map

| User dimension | User stated | Company observed | Fit |
| --- | --- | --- | --- |
| Looking for | | | |
| Location / time-zone | | | |
| Sync vs async | | | |
| Decision pace | | | |
| Hierarchy preference | | | |
| Hours / on-call | | | |
| Domain alignment | | | |

## Outright frictions

{{Hard mismatches. Be blunt. "User wants async, company runs daily standups across 4 time zones — every day-end is a context-switch tax." If there are zero frictions, write "none identified — investigate further" rather than padding.}}

## Soft frictions / things to probe in interview

{{Open questions that would resolve "uncertain" verdicts. Frame as questions the user should ask.}}

1. ?
2. ?
3. ?

## Sources

1. {{url}} — {{what was sourced}}
2. ...
3. Workspace ground-truth.md — {{section}} (last modified {{date}})

## Guardrails

- Ground-truth is privileged: don't override it with company marketing.
- If ground-truth is missing a relevant constraint, ask the user once during the brief — don't fabricate a value.
- "Strong fit" verdicts require at least one specific evidence pair (user dimension → company signal). Generic "looks like a great culture" is not allowed.
- "Poor fit" verdicts are valuable; do not soften them.

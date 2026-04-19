# {{WORKSPACE_NAME}}

Career planning + continuous professional development (CPD) workspace, provisioned from the `career` plugin.

## Profile

- **Role / discipline:** {{ROLE}}
- **Career stage:** {{STAGE}}
- **Headline goal (3-year):** {{HEADLINE_GOAL}}

Full profile in `context/profile.md`.

## Folder layout

| Folder | Purpose |
|---|---|
| `goals/` | Career goals — long-term (3-10 yr), medium-term (1-3 yr), short-term (12 mo). One file per goal. |
| `skills/` | `inventory.md` (current state), `target-profile.md` (what the goals require), `gaps.md` (the diff). |
| `cpd-log/` | Dated learning-activity entries — one file per month (`YYYY-MM.md`). |
| `courses/` | One file per course considered/enrolled/completed. Platform, cost, time, skills targeted. |
| `certifications/` | Certs held, in-progress, planned. Exam + renewal dates. |
| `conferences/` | Events on the radar, attended, planned. CFP deadlines, cost, learnings. |
| `reviews/` | Quarterly + annual self-reviews. Plan vs actual. |
| `context/` | Profile, learning preferences, employer CPD policy. |
| `outputs/` | Generated artefacts — narratives, gap analyses, learning plans. |

## Conventions

- **The skill gap drives the plan.** Cross-reference new learning against `skills/gaps.md`. Activities that don't close a gap or serve a goal are `interest-driven` — still valid, just honest.
- **Log everything, even the small stuff.** A 30-min talk at lunch belongs in `cpd-log/` with date and one-line takeaway. Patterns emerge from consistent capture.
- **Quarterly reviews are non-negotiable.** Every 3 months. Goals not progressing get adjusted, deferred, or retired — with the reason recorded.
- **Cost matters.** Time + money + expected return, for every course/cert/conference.

## Plugin commands (globally installed)

Provisioning: `/career:new-workspace`.

Primitives: `/career:log-role`, `/career:compare-offer`, `/career:salary-benchmark` (career moves).

This variant: `/career:onboard`, `/career:log-cpd`, `/career:skill-gap`, `/career:find-courses`, `/career:find-conferences`, `/career:quarterly-review`.

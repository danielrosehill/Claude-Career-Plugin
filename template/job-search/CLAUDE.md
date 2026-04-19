# {{WORKSPACE_NAME}}

Job-search workspace, provisioned from the `career` plugin.

## Profile

- **Current role:** {{ROLE}}
- **Stage:** {{STAGE}}
- **Target titles:** {{HEADLINE_GOAL}}

Full profile in `context/profile.md`. Must-haves, nice-to-haves, and dealbreakers in `context/constraints.md`.

## Folder layout

| Folder | Purpose |
|---|---|
| `user-context/` | Source of truth — resume, voice notes, personal-brand docs. Prefer many small markdown files. |
| `inputs/` | Raw material the user drops in. `inputs/data/` for JDs, PDFs, CSVs. `inputs/prompt-queue/` for pending/running prompts. |
| `data/processes.json` | Application pipeline database. Managed by the `application-data-manager` agent. |
| `outputs/analysis/` | Research artefacts — company reports, interview-prep briefs, market research. |
| `outputs/cover-letters/` | Generated cover letters. |
| `outputs/resume-versions/` | Tailored resume variants. Base resume stays in `user-context/`. |
| `outputs/personal-branding/` | Bios, positioning, LinkedIn copy. |
| `context/` | Profile, constraints, target companies (allow/block). |

## Conventions

- **Base resume is the source of truth.** Tailored variants live in `outputs/resume-versions/`. Never overwrite the base.
- **Every application gets an event trail.** Every status change goes through `/career:track-application` — don't hand-edit `data/processes.json` unless correcting malformed data.
- **White hat only** on compensation and company research.
- **This workspace is sensitive.** Default to a private repo. Don't commit interview feedback, salary numbers, or personal contact info you wouldn't share publicly.

## Plugin commands (globally installed)

Provisioning: `/career:new-workspace`.

Primitives: `/career:log-role`, `/career:track-application`, `/career:compare-offer`, `/career:salary-benchmark`.

This variant: `/career:onboard`, `/career:pipeline-status`, `/career:research-company`, `/career:tailor-resume`, `/career:cover-letter`, `/career:interview-prep`.

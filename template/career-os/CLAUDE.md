# {{PROJECT_NAME}} — Career-OS workspace

This is a Career-OS workspace: a unified employment + freelance/consulting context store consumed by the `career` Claude Code plugin.

The plugin commands are global (`/career:*`). This directory holds **only data** — your ground-truth profile, research outputs, outreach log, drafts, templates, and cache.

## Variant

`career-os` — unified workspace covering both job-search and consulting / freelance discovery. Reads `ground-truth.md` as source of truth.

## Key files

- `ground-truth.md` — single source of truth (who I am, looking-for, constraints, domains, companies engaged). Edit here; downstream context cascades.
- `resume/` — master CV + per-application variants.
- `templates/` — outreach templates (cold-pitch, warm-followup, job-followup, interview-followup, consulting-pitch).
- `snippets/` — reusable paragraphs.
- `branding/` — links and copies of public profiles (website, GitHub, HF, LinkedIn).
- `crm/companies.md` — markdown table of every company on radar.
- `crm/outreach.md` — append-only outreach log. The spine — every recommendation reads this first.
- `crm/opportunities.md` — pipeline view derived from outreach.md.
- `research/companies/<slug>/` — research brief outputs.
- `domain-notes/` — per-domain enumerations.
- `cache/` — salary research, market data (run-once-and-cache). Gitignored.
- `inbox/` — voice-capture landing zone before routing. Optionally gitignored.

## Plugin commands

Foundation:
- `/career:onboard` — first-run setup; populates ground-truth + config.
- `/career:ground-truth` — read/edit the source-of-truth doc.
- `/career:install-companion-plugins` — offer to install transcription, schedule, email, social, decision-eval, pinecone wiring.

Research:
- `/career:research-company <name>` — orchestrator over six briefs.
- `/career:research-brief <name> <brief>` — single brief deep-dive.

Outreach (Stage 3+):
- `/career:outreach {find|draft|send|log|status} ...`

Discovery + recommendation (Stage 4+):
- `/career:discover {by-domain|like|remote-friendly|by-geography|chunked} ...`
- `/career:suggest {companies|roles|skills|projects|consulting-packages} ...`

Voice + meeting prep (Stage 5+):
- `/career:capture <audio-or-text>`
- `/career:meeting-prep <company>`

Ecosystem (Stage 6+):
- `/career:ecosystem {conferences|communities|boards|hackathons|investors|linkedin-import}`

Planning (Stage 7+):
- `/career:plan week` — weekly plan.
- `/career:plan log` — weekly retrospective.
- `/career:reconcile` — surface discrepancies between ground-truth and CRM.

## Configuration

Plugin-level config lives at `~/.config/career-os/config.json` and points back to this workspace via `WORKING_FOLDER`. Edit the config to switch context-store, email sender, or CRM adapter.

## Notes

- This workspace contains **personal data**. If you publish it, redact ground-truth, crm, and drafts.
- The plugin never writes outside this workspace except for the global config file.
- Re-running any command is idempotent; overwrites are confirmed.

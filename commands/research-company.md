---
description: Orchestrate a full six-brief deep-dive on a target company — overview, financials, recruitment, remote-friendliness, glassdoor, cultural-fit — then synthesise a top-level SUMMARY with best/worst signals, candidate interview questions, and a fit-with-ground-truth verdict.
---

# /career:research-company

Six modular briefs run in parallel, then a top-level synthesis. Use when the user is seriously considering a company as employer or client. For a single-brief deep-dive, use `/career:research-brief` instead.

## Arguments

`$ARGUMENTS`:
- **Required**: `<company_name>` — first positional.
- Optional: `--lens=<employer|client|partner>` (default `employer`).
- Optional: `--slug=<slug>` (default: kebab-case of name).
- Optional: `--briefs=<comma-list>` — limit to a subset of: `overview,financials,recruitment,remote,glassdoor,cultural`. Default: all six.
- Optional: `--refresh` — overwrite existing brief files without prompting.
- Optional: `--no-summary` — produce briefs only, skip SUMMARY synthesis.

## Procedure

### 1. Resolve config

`~/.config/career-os/config.json` → `WORKING_FOLDER`, `USER_LOCATION`. If config is missing, bail with instructions to run `/career:onboard`.

### 2. Decide brief set

Default = all six. If `--briefs=` provided, validate and use that subset. If `cultural` is in the set, ensure `ground-truth.md` exists and is populated; if not, prompt the user to run `/career:ground-truth` first or drop `cultural` from this run.

### 3. Run briefs in parallel

Invoke each selected brief skill in parallel (single message, multiple skill invocations):

- `brief-company-overview <name> --lens=<lens> --slug=<slug>`
- `brief-company-financials <name> --slug=<slug>`
- `brief-recruitment-profile <name> --slug=<slug>`
- `brief-remote-friendliness <name> --slug=<slug>` (passes `USER_LOCATION` from config)
- `brief-glassdoor-signal <name> --slug=<slug>`
- `brief-cultural-fit <name> --lens=<lens> --slug=<slug>`

Pass `--refresh` through if set.

### 4. Synthesise SUMMARY (unless `--no-summary`)

Read all written briefs from `${WORKING_FOLDER}/research/companies/${slug}/`. Produce `${WORKING_FOLDER}/research/companies/${slug}/SUMMARY.md` with:

```markdown
# {{company_name}} — Synthesis

> Lens: {{lens}}
> Briefs read: {{list}}
> Synthesised: {{date}}

## Verdict

- **Recommended next action**: `apply | reach-out | pass | keep-watching`
- **Fit with ground-truth**: `strong | likely | uncertain | poor`
- **Confidence**: `high | medium | low`
- **Headline**: one sentence.

## 3 best signals

1. ... (cite which brief)
2. ...
3. ...

## 3 worst signals

1. ... (cite which brief)
2. ...
3. ...

## 5 candidate interview questions

1. ?
2. ?
3. ?
4. ?
5. ?

(Pull soft-friction probes from cultural-fit; pull process unknowns from recruitment-profile; pull strategic unknowns from overview.)

## Open unknowns

- {{Fields that came back `unknown` or `not disclosed` across briefs that would change the verdict if known.}}

## Source briefs

- [Company Overview](company-overview.md)
- [Financials](company-financials.md)
- [Recruitment Profile](recruitment-profile.md)
- [Remote Friendliness](remote-friendliness.md)
- [Glassdoor Signal](glassdoor-signal.md)
- [Cultural Fit](cultural-fit.md)
```

Decision rules for "Recommended next action":
- `apply` — strong/likely cultural fit, remote-friendliness `yes` or N/A, no major financial red flag, hiring activity present.
- `reach-out` — strong fit but no current open role; warm outreach via consulting-pitch / cold-pitch makes sense.
- `pass` — `poor` cultural fit, OR remote-friendliness `no`, OR explicit constraint violation from ground-truth.
- `keep-watching` — uncertain fit OR thin signals OR cold pipeline; revisit in 60–90 days.

### 5. Update CRM

Append/update the company row in `${WORKING_FOLDER}/crm/companies.md`:
- `status` → `researched` (unless already `engaged` / `active` / `paused`).
- `last-touch` → today.
- `research-brief` → `full` (replaces individual `+overview`/`+financials`/etc tags).

### 6. Print summary

```
research-company complete: <slug>
briefs written: <n>/<requested>
verdict: <next-action> (<fit>)
file: research/companies/<slug>/SUMMARY.md
```

## Failure modes

- **One brief fails** — report which, continue with the others, mark the SUMMARY as `## Status: partial`.
- **Ground-truth missing** — drop `cultural` from the run, note in SUMMARY.
- **Network failure mid-run** — partial briefs are written; user can re-run with `--briefs=` to fill the gaps.

## Idempotency

- Re-run without `--refresh` → each brief prompts before overwrite.
- Re-run with `--refresh` → silently overwrites all briefs and SUMMARY.

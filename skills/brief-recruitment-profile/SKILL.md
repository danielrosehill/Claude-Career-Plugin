---
name: brief-recruitment-profile
description: Produce a Recruitment Profile research brief — careers page, open roles (filtered by target function if provided), LinkedIn presence, headcount trend, hiring signals, recruiter activity, and known interview-process patterns from public sources. Use when the user wants to assess whether a company is actually hiring people like them, vs. posting ghost roles. Reads templates/research-briefs/recruitment-profile.md and writes to <WORKING_FOLDER>/research/companies/<slug>/recruitment-profile.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Recruitment Profile

How and whether they're actually hiring. Distinguishes live activity from boilerplate "we're always hiring" pages.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>`.
- Optional: `--slug=<slug>`.
- Optional: `--function=<engineering|product|design|...>` — narrows open-role analysis.

## Procedure

### 1. Resolve config + paths

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. Output: `${WORKING_FOLDER}/research/companies/${slug}/recruitment-profile.md`.

### 2. Read template

`${PLUGIN_ROOT}/templates/research-briefs/recruitment-profile.md`.

### 3. Surfaces to gather

- Company careers page.
- LinkedIn company page (employee count, follower count, recent activity).
- Glassdoor "Interviews" tab — for process patterns.
- Indeed / AngelList / Wellfound listings.
- Levels.fyi — for compensation context (cross-link, don't duplicate Glassdoor brief).
- LinkedIn search for "<company> recruiter" — recent activity.

### 4. Open roles analysis

For each open posting in the target function (or all, if no function):
- Title, function, location, remote eligibility, posted date, link.
- Flag postings >60 days old as `ghost-suspect`.

Aggregate:
- Total open roles.
- Roles per function.
- Geography distribution.

### 5. Headcount trend

Pull LinkedIn employee count for the latest 1–4 snapshots from Wayback Machine if needed. Compute direction + magnitude over the available period.

### 6. Hiring signals

Recent senior hires, recent senior departures (signal-bearing only — not every leaver), recent layoffs (date + scope + statement), recent recruiter posts on LinkedIn.

### 7. Process notes

From Glassdoor "Interviews" tab — summarise typical stages, timeline, takehome y/n, pay-band transparency. Quote with reviewer date.

### 8. Write output

Standard pattern (see other briefs).

### 9. Update CRM

Append `+recruitment` to the row.

## Companion delegation

If `social-feedback` companion is installed, delegate sentiment cross-checks ("are people on Reddit/HN talking about this company's hiring process?") via `social-feedback:check-discourse`. Otherwise skip.

## Guardrails

- Distinguish live posting from ghost role.
- Glassdoor anonymous reviews are signal, not fact.
- LinkedIn employee count is approximate — note snapshot date.
- Don't list every junior departure; only signal-bearing senior moves.

## Failure modes

- **No public LinkedIn page** → skip headcount trend, mark `unknown`, continue.
- **Careers page blocks fetch** → use search engine cached snippets; mark fields with `(cached)`.

## Idempotency

- Re-run prompts before overwrite.
- `--refresh` silently overwrites.

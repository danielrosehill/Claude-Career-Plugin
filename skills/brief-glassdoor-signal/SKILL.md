---
name: brief-glassdoor-signal
description: Produce a Glassdoor / employee-sentiment research brief — overall ratings, sub-ratings, tenure data with red-flag detection scoped to the relevant team, compensation data, theme analysis of top complaints and positives, recent direction inflections. Aggregate sentiment, not a recap of individual reviews. Use to gauge whether public employee discourse aligns or contradicts the company's outward image. Reads templates/research-briefs/glassdoor-signal.md and writes to <WORKING_FOLDER>/research/companies/<slug>/glassdoor-signal.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Glassdoor Signal

Aggregate sentiment, *not* a list of individual reviews. The point is patterns and inflections.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>`.
- Optional: `--slug=<slug>`.
- Optional: `--team=<engineering|sales|product|...>` — scope tenure red flags + theme analysis to a team.

## Procedure

### 1. Resolve config + paths

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. Output: `${WORKING_FOLDER}/research/companies/${slug}/glassdoor-signal.md`.

### 2. Read template

`${PLUGIN_ROOT}/templates/research-briefs/glassdoor-signal.md`.

### 3. Gather

- Glassdoor company page (rating, sub-ratings, review count, CEO approval, recommend-to-friend).
- Glassdoor reviews list (latest 30 + historical sample).
- Levels.fyi / Blind / TeamBlind / Comparably for compensation cross-checks.

### 4. Compute headline numbers

From the company page directly. Trend: compare current rating to last 12 months if Wayback snapshots available.

### 5. Tenure red-flag check

- Compute median tenure overall.
- If `--team` provided, scope to that team's reviewers (use job-title field).
- **Only flag sub-1-year medians scoped to a relevant team or to the whole company at <50 headcount.** Sub-1-year medians at company-wide scale on a 5000-person org are noise.
- Quote 1–2 representative tenure-red-flag reviews if you flag.

### 6. Theme analysis

For the latest 30 reviews:
- Cluster negative reviews into 3–5 themes by frequency.
- Cluster positive reviews into 3–5 themes by frequency.
- For each theme, quote 1–2 representative reviewers verbatim with role + date.
- Look for *patterns*. Single-instance complaints ≠ themes.

### 7. Inflection detection

If reviews show a clear before/after — leadership change, layoffs, acquisition — note it with one quote on each side.

### 8. Compensation data

Pull stratified data (role × geography) from Levels / Glassdoor / Blind. Always cite source. Self-reports get "(self-reported)" tag.

### 9. Write output + update CRM

Append `+glassdoor` to the row.

## Companion delegation

If `social-feedback` companion is installed, cross-check on Reddit / HN / Blind — `social-feedback:check-discourse <company> employee sentiment`. Use to validate or qualify Glassdoor themes.

## Guardrails

- **Don't over-index on outliers.** Single 1-star review with personal grievance is not a theme.
- **Tenure red flags are scoped.** See decision rule above.
- **No invented quotes.** Every blockquote is verbatim from a real review with date.
- Self-reported comp data is tagged.
- Acknowledge low review volume (<20) explicitly — high-variance.

## Failure modes

- **Glassdoor page missing or fetch blocked** → fall back to Indeed reviews + Comparably; mark with `(non-Glassdoor source)`.
- **Review volume too low (<10)** → write `## Status: low-signal` and avoid theme analysis.

## Idempotency

- Re-run prompts before overwrite.
- `--refresh` silently overwrites.

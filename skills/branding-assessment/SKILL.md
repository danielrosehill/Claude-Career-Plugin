---
name: branding-assessment
description: Audit the user's public brand surfaces — personal site, GitHub, Hugging Face, LinkedIn — against ground-truth direction. Surfaces drift (site says X, ground-truth says Y) and gaps (no project shipped in domain Z). Produces an actionable update list.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Branding Assessment

Surface mismatch between what the user *says* they're focused on (ground-truth) and what their public presence *shows*. Hiring managers and prospects look at the surfaces; ground-truth is invisible.

This is a diagnostic skill — it produces a list of edits, doesn't apply them.

## Inputs

`$ARGUMENTS`:
- Optional: `--surface=<website|github|hugging-face|linkedin|all>` — default `all`.
- Optional: `--target=<role|domain|consulting>` — what the brand should be optimised for. Defaults to ground-truth's primary direction.
- Optional: `--depth=<quick|deep>` — default `quick` (top-of-page only). `deep` walks into individual repos / projects.

## Prerequisites

- `ground-truth.md` populated.
- `branding/config.yaml` (or workspace config keys):
  - `WEBSITE_URL`
  - `GITHUB_HANDLE`
  - `HUGGING_FACE_HANDLE`
  - `LINKEDIN_URL`

If config missing, prompt for the URLs and offer to write `branding/config.yaml`.

## Procedure

### 1. Load context

- ground-truth.md (direction, domains-of-interest, freelance-brand, hard-constraints).
- branding config.

### 2. Fetch each surface

For each surface in scope:

- **Website** — fetch home page + about page. Extract: tagline, primary positioning, recent posts/projects, calls to action.
- **GitHub** — profile bio, pinned repos, recent commit cadence, language distribution. `--depth=deep` reads the README of pinned repos.
- **Hugging Face** — profile bio, models/datasets/spaces, recent activity.
- **LinkedIn** — headline, about, recent activity (only what's publicly accessible without auth).

If a surface fetch fails, note it; don't bail.

### 3. Compare to ground-truth

Per surface, score on three axes:

- **Direction match** — does the surface's positioning match ground-truth's primary direction?
  - aligned / partial / drifted / silent.
- **Domain coverage** — are ground-truth's domains-of-interest visible?
  - well-covered / partial / missing.
- **Freshness** — last update vs. cadence target.
  - fresh / stale (60–180 days) / very stale (>180 days).

### 4. Surface findings

Categorise:
- **Critical** — direction drifted (e.g. ground-truth = "AI policy consulting", website tagline = "full-stack dev for hire").
- **Gaps** — domain in ground-truth but no shipped artifact in that domain on any surface.
- **Stale** — surface hasn't moved in months while user has shipped elsewhere.
- **Quick wins** — bio one-liners, pinned-repo selection, headline edits.

### 5. Write report

`${WORKING_FOLDER}/branding/assessment-<date>.md`:

```markdown
# Branding assessment — {{date}}

**Target:** {{role|domain|consulting}}
**Surfaces:** {{list}}

## Direction match

| surface | score | evidence |
| --- | --- | --- |
| website | partial | tagline says X, ground-truth says Y |
| ...

## Domain coverage

(per domain → which surfaces show shipped work)

## Stale surfaces

...

## Recommended edits (ordered)

1. **{{surface}}** — {{specific edit}}. Why: {{from-ground-truth}}.
2. ...

## Gaps to ship

(domains in ground-truth with no public artifact — candidate `/career:suggest projects` queries)
```

### 6. Print summary

```
report: branding/assessment-<date>.md
critical: <n>  gaps: <n>  stale: <n>  quick-wins: <n>

next:
  critical → edit the offending surface
  gaps → /career:suggest projects --domain=<missing>
```

## Guardrails

- Recommendations only; never edits external surfaces.
- Don't moralise. "Stale GitHub" is a fact, not a failure.
- Specific evidence required for every finding (a quoted line, a missing-pinned-repo note). No generic "your brand is unclear."

## Failure modes

- Surface unreachable (auth wall, 404) → note and continue.
- ground-truth absent → bail; assessment is meaningless without an anchor.

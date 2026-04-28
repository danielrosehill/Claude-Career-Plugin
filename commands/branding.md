---
description: Audit public brand surfaces — website, GitHub, Hugging Face, LinkedIn — against ground-truth direction. Surfaces drift, gaps, and stale surfaces. Recommendations only.
---

# /career:branding

Single-skill command. Runs the `branding-assessment` skill.

## Usage

```
/career:branding [--surface=<website|github|hugging-face|linkedin|all>] [--target=<role|domain|consulting>] [--depth=<quick|deep>]
```

## What it does

- Fetches each public surface and compares positioning against `ground-truth.md`.
- Scores each on direction match, domain coverage, freshness.
- Categorises findings: critical / gaps / stale / quick-wins.
- Writes `branding/assessment-<date>.md`. Never edits external surfaces.

## Prerequisites

- `ground-truth.md` populated.
- `branding/config.yaml` (or workspace config) with `WEBSITE_URL`, `GITHUB_HANDLE`, `HUGGING_FACE_HANDLE`, `LINKEDIN_URL`.

## Notes

- Recommendations only — the user owns the edits.
- Domains-of-interest with no shipped artifact are surfaced as candidates for `/career:suggest projects`.

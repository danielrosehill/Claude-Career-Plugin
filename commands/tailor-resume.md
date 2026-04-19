---
description: Generate a resume variant tailored to a specific role/JD, preserving factual accuracy.
---

# /career:tailor-resume

Produce a tailored resume version for a specific application. Never invents experience.

## Steps

### 1. Inputs

- Base resume: `user-context/resume.md` (or wherever the workspace stores it — ask if unclear).
- Job description: `$ARGUMENTS` may be a file path, URL, or a tracked slug from `data/processes.json`. Otherwise ask.

### 2. Analyse the JD

Extract: must-have skills, nice-to-haves, keywords, tone, seniority signals, named technologies.

### 3. Tailor

Rewrite the base resume so that:

- Bullets surface the experience most relevant to the JD.
- Phrasing mirrors the JD's keywords **where truthful**.
- Irrelevant content is cut or compressed.
- **No invented experience.** If the user lacks a JD must-have, leave it out; don't fabricate.

### 4. Save

Write to `outputs/resume-versions/<date>-<company>-<role-slug>.md`. Include a top-of-file note summarising the tailoring choices so the user can review.

### 5. Report

Show the diff summary (cuts, rewrites, reorderings) and any JD requirements the user lacks — those are gaps worth addressing before applying.

---
description: Generate a cover letter for a specific application, grounded in the user's actual experience.
---

# /career:cover-letter

Draft a cover letter tailored to a tracked role/JD.

## Steps

### 1. Inputs

- Base resume and profile: `user-context/resume.md`, `context/profile.md`.
- Target role: `$ARGUMENTS` — slug, company, or path to JD. Otherwise ask.

### 2. Draft

Structure:

1. **Hook** — why this company, specifically. One concrete signal (product, recent news, team). No generic flattery.
2. **Fit** — 2-3 experiences from the resume that map to the JD's must-haves. Evidence, not claims.
3. **Value** — one short paragraph on what the user would bring in the first 90 days.
4. **Close** — availability + contact.

Keep to one page. Match the tone of the JD (formal vs. casual).

### 3. Save

`outputs/cover-letters/<date>-<company>-<role-slug>.md`.

### 4. Report

Show any JD expectations the letter doesn't address so the user can decide whether to add them or let them slide.

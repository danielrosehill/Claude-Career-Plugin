---
description: Surface candidate courses that address the user's top skill gaps.
---

# /career:find-courses

Suggest courses to close the top critical gaps in `skills/gaps.md`.

## Steps

1. Read `skills/gaps.md`. If missing, tell the user to run `/career:skill-gap` first.
2. Pick the top 3 critical gaps (or whatever `$ARGUMENTS` specifies).
3. For each gap, propose 2-4 course candidates across mixed platforms (Coursera, edX, Udemy, O'Reilly, university-run, vendor-native certs, free MOOCs). Prefer courses with recent reviews and clear outcomes over brand.
4. For each candidate, capture: platform, URL, cost, time commitment, prerequisite, outcome (cert / skill), reviewer-reported quality signal.
5. Save to `courses/<gap-slug>.md` (one file per gap) or append if it exists.
6. Present a short shortlist. Don't enrol on the user's behalf — they decide.

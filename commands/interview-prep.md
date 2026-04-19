---
description: Build an interview prep brief for a specific round — likely questions, answers anchored in the user's experience, and company-specific research.
---

# /career:interview-prep

Prepare for a specific interview round with material grounded in the user's resume and the company research brief.

## Steps

### 1. Inputs

- Application: `$ARGUMENTS` — slug, or prompt to pick from `data/processes.json` entries in `interviewing` / `onsite` status.
- Round: screen / tech / system-design / behavioural / leadership / onsite-panel. Ask if unclear.
- Company brief: `outputs/analysis/company-reports/<slug>.md` (run `/career:research-company` first if missing).

### 2. Generate questions

Based on role, seniority, round, and JD:

- Likely technical questions (role-specific).
- Likely behavioural questions (STAR-able from the user's resume).
- Company-specific questions (e.g. product knowledge, recent news).
- Questions the user should ask the interviewer.

### 3. Draft anchored answers

For each likely question, sketch an answer using experience from the resume. Mark `[NEEDS STORY]` where the user should prepare a concrete STAR example. Do not invent.

### 4. Save

`outputs/analysis/interview-prep/<date>-<slug>-<round>.md`.

### 5. Report

List the top 5 questions to rehearse out loud and the top 3 `[NEEDS STORY]` gaps.

---
description: Register a new role of interest — for tracking, applying, or salary benchmarking.
---

# /log-role

Record a role in the workspace so later commands (`/compare-offer`, `/track-application`, `/salary-benchmark`) can reference it.

## Steps

### 1. Capture the role

If the user supplied details inline via `$ARGUMENTS`, parse them. Otherwise ask (AskUserQuestion + free-text):

- **Company:** organisation name
- **Title:** role/job title
- **Level:** (if known) IC-level, manager band, etc.
- **Location / remote policy**
- **Source:** where it was found (LinkedIn, referral, company site, recruiter, etc.)
- **Status:** `interested` / `applied` / `interviewing` / `offer` / `rejected` / `withdrawn` (default `interested`)
- **Link:** JD / posting URL if available
- **Notes:** anything distinctive (team, product, comp hints, timeline)

### 2. Decide where to store it

The workspace variant determines the destination (read `CLAUDE.md` if unsure):

- **job-search** → append to `data/processes.json` (pipeline DB). If the file doesn't exist yet, create it with an empty array.
- **salary-research** → create `analysis/roles/<slug>.md` using the role profile template.
- **career-planning** → create `goals/roles/<slug>.md` if the role ties to a goal; otherwise append to `context/roles-of-interest.md`.

### 3. Write the entry

Use today's date. Slug = `<company>-<title>` kebab-cased. Preserve any existing entries — never overwrite.

### 4. Confirm

Print a one-line confirmation with the path written and the slug, so follow-up commands can address it directly.

---
description: Quick-log a CPD (continuing professional development) activity — article, talk, book, course session, mentoring, etc.
---

# /career:log-cpd

Friction-free capture of a learning activity. Patterns only emerge from consistent logging.

## Steps

### 1. Capture

Parse `$ARGUMENTS` if supplied (e.g. `/career:log-cpd "watched Helmer 7 Powers talk, 60 min"`). Otherwise ask:

- **Type:** article / talk-or-video / book / book-chapter / podcast-episode / course-session / mentoring / pairing / project-work / conference-session / other
- **Title**
- **Source / link**
- **Duration (min)**
- **One-line takeaway**

### 2. Tag against goals/gaps (optional)

If `skills/gaps.md` or `goals/` exist, suggest the most relevant tag. Let the user accept, change, or skip. If skipped, tag `interest-driven`.

### 3. Write

Append to `cpd-log/YYYY-MM.md`:

```markdown
## YYYY-MM-DD — <title>

- **Type:** <type>
- **Source:** <link>
- **Duration:** <minutes>
- **Tagged:** <gap/goal/interest-driven>
- **Takeaway:** <one line>
```

Create the monthly file with a `# YYYY Month` header if missing.

### 4. Inventory bump (optional)

If the activity meaningfully closed a gap, ask whether to bump the matching row in `skills/inventory.md`.

### 5. Confirm

One line. This should feel cheap to run.

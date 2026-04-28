---
name: brief-remote-friendliness
description: Produce an honest Remote Friendliness research brief — does the company actually hire from the user's location, or is the "remote-first" claim marketing? Cross-references stated policy against live job postings, time-zone requirements, hiring entities, and existing employees in the user's region. Critical for non-US/EU users. Reads templates/research-briefs/remote-friendliness.md and writes to <WORKING_FOLDER>/research/companies/<slug>/remote-friendliness.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Remote Friendliness

The brief that reads against marketing copy. Many "remote-first" companies will not in practice hire someone outside US W-2 / EU EOR jurisdictions. This skill exists to surface that before the user invests time in an application.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>`.
- Optional: `--slug=<slug>`.
- Optional: `--user-location=<location>` — overrides `USER_LOCATION` from config.

## Procedure

### 1. Resolve config + paths

Read `~/.config/career-os/config.json`. Get `WORKING_FOLDER` and `USER_LOCATION`. If `USER_LOCATION` is missing and not provided, prompt — this brief is meaningless without it.

Output: `${WORKING_FOLDER}/research/companies/${slug}/remote-friendliness.md`.

### 2. Read template

`${PLUGIN_ROOT}/templates/research-briefs/remote-friendliness.md`.

### 3. Gather stated policy

Direct quotes from:
- Careers page "remote" / "how we work" sections.
- Engineering blog posts about distributed work.
- CEO/founder public statements (podcasts, posts, LinkedIn).

Capture URL for each quote. **No paraphrase** — quote verbatim.

### 4. Pull live postings

Fetch every recent job posting (last 90 days). Per posting record:
- Source URL.
- Stated location.
- Stated time-zone constraint.
- Stated entity / hiring jurisdiction (W-2, contractor, EOR-only).

### 5. Cross-reference against `{{user_location}}`

For the user's location, determine:
- Has any live posting accepted candidates from this location? (most decisive signal)
- Are there time-zone overlap requirements that exclude it?
- Does the company have a known hiring entity (Deel, Remote.com, etc.) that operates there?
- Are there existing employees on LinkedIn in the user's location? (≤5 examples with role + tenure.)

### 6. Write the verdict

Top of the file:
- **Eligible for {{user_location}}?** `yes | conditional | no | unclear`
- **Confidence**: `high | medium | low`
- **Headline reason**: one sentence.

Decision rules:
- `yes` requires: at least one live posting open to the user's location AND existing employees there OR explicit policy statement covering the location.
- `no` is the default if no live posting matches the user's location, regardless of marketing copy.
- `conditional` for time-zone overlap requirements that the user could meet only with shifted hours.
- `unclear` is a valid answer — say so rather than guessing.

### 7. Marketing-vs-reality gap

If careers page says "fully remote" but every recent posting is US-only: write a dedicated section quoting both, with both URLs.

### 8. Write output + update CRM

Standard pattern. Append `+remote` to the company row.

## Guardrails

- **Bias toward honest negatives over hopeful positives.** The user has limited time; false hope wastes it.
- Live postings > marketing copy. Always.
- "Unclear" is acceptable.
- Existing employees in the user's region are the strongest positive signal — surface them when present.

## Failure modes

- **No `USER_LOCATION` config and not provided** → bail.
- **Company has no current open postings** → fall back to last 12 months of postings; flag the cold-pipeline observation.
- **Company explicitly does not have a public careers page (e.g. invite-only / stealth)** → mark `unclear` with low confidence; suggest the user ask directly if they engage.

## Idempotency

- Re-run prompts before overwrite.
- `--refresh` silently overwrites.

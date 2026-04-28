---
name: workspace-organize
description: Flat/local-mode workspace hygiene. Sort loose files into canonical folders (research/, domain-notes/, inbox/, crm/, resume/, snippets/, branding/, cache/), find near-duplicates by name+hash, flag stale inbox entries (>14d), and re-root mis-placed files. Proposes a plan, applies only on confirm. No-op when CONTEXT_STORE != markdown.
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash(find *), Bash(sha256sum *), Bash(stat *), Bash(mkdir *), Bash(mv *), Bash(date *), Bash(rg *)
---

# Workspace Organize

## Inputs

`$ARGUMENTS`:
- `--dry-run` (default) — propose only, don't move.
- `--apply` — execute the proposed moves/dedupes after user confirms each batch.
- `--stale-days=N` — inbox staleness threshold (default 14).

## Procedure

### 1. Resolve config

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `CONTEXT_STORE`. Bail early if `CONTEXT_STORE != markdown` with: "vector-mode workspace — run `/career:vector-conflict-check` instead."

### 2. Inventory

`find $WORKING_FOLDER -maxdepth 4 -type f` excluding `.git/`, `cache/`, `node_modules/`. For each file capture: relative path, size, sha256 (cap to files <1 MB; larger files use size+name only), mtime.

### 3. Detect issues

Walk the inventory once and bucket findings:

- **Loose root files** — anything at `WORKING_FOLDER/` that isn't `ground-truth.md`, `CLAUDE.md`, `README.md`, `.gitignore`. Propose a target folder by content sniff (see table below).
- **Mis-rooted files** — file lives in folder X but content matches Y's shape (e.g. a research brief sitting in `domain-notes/`). Detect via filename pattern + first 200 chars.
- **Duplicates** — same sha256 in two paths. Keep the one in the canonical folder; propose deleting the other.
- **Near-duplicates** — same basename across different folders OR levenshtein <3 on basename. Show side-by-side; let user decide.
- **Stale inbox** — `inbox/*.md` with `status: pending-route` AND mtime older than `--stale-days`. Propose archiving to `inbox/_stale/`.
- **Empty folders** — propose removal.

### 4. Routing table (content sniff → target)

| Signal | Target |
| --- | --- |
| frontmatter `classification:` or filename `YYYY-MM-DD-HHMM.md` | `inbox/` |
| heading `# Outreach log` or rows like `\| company \| date \|` | `crm/` |
| heading `# Company Overview` / `# Glassdoor Signal` / `# Cultural Fit` etc. | `research/companies/<slug>/` |
| heading `# Communities` / `# Conferences` / `# Hackathons` etc. | `domain-notes/<slug>/` |
| filename `cv*`, `resume*` | `resume/` |
| filename `cold-pitch*`, `*-followup*` | `templates/` |
| pdf, png, jpg with no clear mate | `branding/` if filename suggests a profile asset, else flag as `unrouted` |

If no signal matches, mark `unrouted` and surface for user decision.

### 5. Propose

Print a per-bucket summary (counts) and a per-file table for moves/dedupes:

```
LOOSE ROOT (3)
  notes-from-call.md            → inbox/2026-04-29-notes-from-call.md
  acme-research.md              → research/companies/acme/company-overview.md
  cv-final-final.pdf            → resume/cv-final-final.pdf

DUPLICATES (2)
  resume/cv.pdf  ==  branding/cv.pdf      keep resume/cv.pdf, delete branding copy

STALE INBOX (4 older than 14d)
  inbox/2026-03-12-1100.md      → inbox/_stale/

UNROUTED (1)
  weird-file.txt                — needs your call
```

If `--dry-run`, stop here.

### 6. Apply (only if `--apply`)

Confirm per-bucket (`AskUserQuestion`: apply moves / apply dedupes / archive stale / each separately). Execute confirmed batches. Always `mkdir -p` the target. Never overwrite — if target exists, append `-1`, `-2`, etc. and surface for review.

After apply: print a one-line audit row to `${WORKING_FOLDER}/cache/organize-log.md`.

## Idempotency

Re-running on a clean workspace finds nothing and exits with "workspace tidy."

## Failure modes

- **Workspace path missing** → bail with `/career:onboard` instruction.
- **Filesystem error mid-apply** → stop, print what completed, leave the rest as proposals.

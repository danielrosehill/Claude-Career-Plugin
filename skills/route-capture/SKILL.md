---
name: route-capture
description: Given a classified inbox entry (from voice-ingest or text input), execute the proposed route — create tasks, update CRM rows, draft follow-ups, propose ground-truth edits. Always confirms each side-effect before committing. Marks the inbox entry routed when done.
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Route Capture

The "do something with the inbox entry" half of the capture pipeline. Reads the classified inbox file, executes the route appropriate to its classification, confirms each side-effect.

## Inputs

`$ARGUMENTS`:
- **Required**: `<inbox-path>` — path to an inbox entry (relative to `WORKING_FOLDER` or absolute).
- Optional: `--reclassify=<class>` — override the inbox file's classification.
- Optional: `--dry-run` — show what would happen, don't write.

### Examples

```
/career:capture route inbox/2026-04-28-1430.md
/career:capture route inbox/2026-04-28-1430.md --reclassify=retrospective
/career:capture route inbox/2026-04-28-1430.md --dry-run
```

## Procedure

### 1. Load + validate

Read the inbox file. Check frontmatter `status` is `pending-route`. If `routed`, ask whether to re-route (rare).

Resolve effective classification: `--reclassify` if set, else frontmatter `classification`.

### 2. Branch by classification

#### a) `quick-note`

Read transcript. Extract:
- A task (if action-shaped). Propose `schedule-manager:create-task` with title + optional due date inferred from transcript.
- A company/contact mention (if present). Propose append to `crm/companies.md` (status=watching, lens=employer by default — user can edit).

Show each proposed write as a diff. Confirm per-item (multi-step AskUserQuestion or numbered y/n list). Execute confirmed items.

If `schedule-manager` companion is missing, fall back to writing to `${WORKING_FOLDER}/tasks-inbox.md` (an append-only file the user can later sync manually).

#### b) `retrospective`

Read transcript. Extract:
- Counterparty (person + company).
- Status change (e.g. sent → replied, meeting-booked → meeting-done).
- Next action (often "follow up with X by Y").
- Mentioned future meeting (if any).

Propose:
1. `track-status` row update on `crm/outreach.md` for the counterparty.
2. `draft-outreach` for the follow-up if a follow-up was mentioned.
3. `schedule-manager:create-event` for the future meeting if mentioned.

Each proposal shown as a preview. Confirm per-item.

#### c) `strategic-dump`

Read transcript. Extract topics that look like:
- Ground-truth deltas (new looking-for, changed salary band, new domain-of-interest).
- Research requests ("I should look into X").
- Branding/positioning thoughts.

Propose:
1. Append a `## Strategic notes — {{date}}` section to `${WORKING_FOLDER}/strategic-dumps.md` with the cleaned transcript.
2. For each detected ground-truth delta: a per-section diff against current `ground-truth.md`. **Don't write** — surface the diff and recommend `/career:ground-truth edit-section` for the affected sections.
3. For each research request: append to `${WORKING_FOLDER}/research-queue.md` (append-only).

#### d) `unclear`

Don't route. Update frontmatter `status: parked` with a one-line note. Tell user: "left in inbox — re-run with `--reclassify=<class>` once intent is clear."

### 3. Update inbox entry

After successful routing, update frontmatter:
```
status: routed
routed-at: {{ISO timestamp}}
route-summary: {{1 line of what was done}}
```

Append a `## Route log` section listing every side-effect:
```
## Route log
- 2026-04-28 14:42 — created task "follow up with Jane" via schedule-manager
- 2026-04-28 14:42 — drafted follow-up: drafts/2026-04-28-snowglobe-jane.md
```

### 4. Print summary

```
inbox: <path>          → routed
side-effects:
  - <effect 1>
  - <effect 2>
...

next:
  - if drafts created: /career:outreach send <draft-path>
  - if ground-truth deltas surfaced: /career:ground-truth edit-section <section>
```

## Guardrails

- **Confirm every external side-effect.** Tasks, calendar events, drafts, CRM rows — all require explicit y per item.
- Never silently overwrite ground-truth from a strategic-dump. The skill *proposes* deltas; the user owns ground-truth edits.
- `--dry-run` must produce identical proposal output to a live run, just without writes.
- Inbox files are not deleted. The frontmatter `status` is the source of truth for routed/unrouted.

## Failure modes

- **Inbox file missing** → bail.
- **Classification not recognised** → bail; ask for `--reclassify`.
- **Companion plugin missing for a route** → fall back gracefully (markdown queue file) and note the fallback in the route log.

---
name: weekly-plan
description: Read crm/outreach.md + crm/opportunities.md + recent recommendations, propose 5–7 actions for the week (mix of research, outreach, follow-up, meetings, ecosystem). Delegates actionable items to schedule-manager:batch-create-tasks and time-blocked work to schedule-manager:batch-create-events. Confirms before any external write.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(date *)
---

# Weekly Plan

Synthesizes the week ahead from the workspace state. Every proposed action ties back to a specific row (an outreach thread waiting for the user, a top-ranked recommendation, a CFP closing, etc).

## Inputs

`$ARGUMENTS`:
- Optional: `--week-of=<YYYY-MM-DD>` — defaults to next Monday.
- Optional: `--target=<n>` (default 6) — number of actions.
- Optional: `--mix=<balanced|outreach-heavy|build-heavy>` (default `balanced`).
- Optional: `--dry-run` — show plan, don't push to schedule-manager.

### Examples

```
/career:plan week
/career:plan week --target=8 --mix=outreach-heavy
/career:plan week --week-of=2026-05-04 --dry-run
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`, `companions.scheduling`. Output: `${WORKING_FOLDER}/plans/$(date -d "${week-of}" +%Y-%m-%d)-week-plan.md`.

### 2. Load state

- `crm/outreach.md` — find rows where status ∈ {sent, replied, meeting-booked} and `next-action` is set or implied.
- `crm/opportunities.md` — current pipeline; what's nearest the close.
- `recommendations/companies-*.md` — most recent file; top picks not yet researched.
- `domain-notes/*/conferences.md` — CFPs closing within the week.
- `inbox/*.md` (status pending-route or parked) — items needing routing.
- `meetings/*-prep.md` — upcoming calls in the week.

### 3. Generate candidate actions

For each candidate:
- One-line action.
- Source row (cite the file + table row).
- Type: `research` | `outreach-draft` | `outreach-send` | `follow-up` | `meeting-prep` | `meeting` | `ecosystem` | `build`.
- Estimated time.
- Skill to invoke (if any).

### 4. Pick a balanced set

Based on `--mix`:
- `balanced`: 1–2 research, 2–3 outreach, 1–2 follow-ups, 0–1 meeting-prep, 0–1 build/ecosystem.
- `outreach-heavy`: 0–1 research, 4–5 outreach, 1–2 follow-ups.
- `build-heavy`: 0–1 outreach, 0–1 follow-up, 3–4 build/ecosystem, 1–2 research.

Aim for `--target` actions. Drop the lowest-leverage candidates if over.

### 5. Write the plan

```markdown
# Week of {{week-of}} — plan

> Generated: {{ISO timestamp}}
> Mix: {{mix}}
> Source state: outreach.md ({{n}} active rows), opportunities.md ({{n}} in pipeline), recommendations/{{latest}}.md

## Actions

### 1. {{title}}
- type: {{type}}
- source: {{file}} row {{n}}
- time: {{est}}
- invoke: `{{command}}`
- why: {{one line}}

### 2. ...

## Time-blocks (calendar)

| day | block | what | duration |

## Tasks (no specific time)

| title | due | tag |

## What I'm explicitly not doing this week (and why)

- {{thing}} — {{reason}}
```

### 6. Push to schedule-manager (unless `--dry-run`)

If `companions.scheduling` is set:
- Time-blocks → `schedule-manager:batch-create-events`.
- Tasks → `schedule-manager:batch-create-tasks`.
- Show diff before push; confirm yes/no/edit.

If companion missing, write tasks/events to `${WORKING_FOLDER}/plans/<date>-todo.md` and tell user.

### 7. Print summary

```
plan: plans/<date>-week-plan.md
actions: <n>
time-blocks: <n>
tasks: <n>
pushed: <n> events / <n> tasks (or "dry-run — nothing pushed")

next: end of week → /career:plan log
```

## Guardrails

- Every action cites a source row. No fabricated "you should also..." entries.
- "What I'm explicitly not doing" forces honesty about scope. If the section is empty, the plan is probably overstuffed.
- Confirm before pushing to schedule-manager. Tasks created in error are a pain to clean up.

## Failure modes

- **Empty workspace** (new user, no outreach yet) → propose week of build/ecosystem instead, surface "no outreach to follow up — start with /career:discover".
- **Schedule-manager missing** → fall back to markdown queue; don't fail the plan.

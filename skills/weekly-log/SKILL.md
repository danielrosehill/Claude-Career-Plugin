---
name: weekly-log
description: End-of-week wrap-up. Reads outreach diffs since last log, reads the week's plan, asks user what worked, what didn't, what's next. Produces a wrap-up doc. Delegates to schedule-manager wrapup-style if available. Saves to <WORKING_FOLDER>/plans/<YYYY-MM-DD>-week-log.md.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(date *), Bash(test *)
---

# Weekly Log

Closes the loop on a week. Diffs the workspace state against the start of the week, asks the user reflective questions, and produces a single document the user can scan in 90 seconds before next week's plan.

## Inputs

`$ARGUMENTS`:
- Optional: `--week-of=<YYYY-MM-DD>` — defaults to current week's Monday.
- Optional: `--quick` — skip reflective Q&A; produce diff-only doc.

### Examples

```
/career:plan log
/career:plan log --week-of=2026-04-21 --quick
```

## Procedure

### 1. Resolve paths

Output: `${WORKING_FOLDER}/plans/$(date -d "${week-of}" +%Y-%m-%d)-week-log.md`.

Find the plan for the same week: `${WORKING_FOLDER}/plans/<same-date>-week-plan.md` if exists.

### 2. Diff workspace state

Compare against the previous week-log if one exists:

- **Outreach diff**: rows added since last log; rows that changed status.
- **Opportunities diff**: stage changes in `crm/opportunities.md`.
- **Briefs added**: new files under `research/companies/` since last log.
- **Recommendations runs**: new files under `recommendations/`.
- **Drafts sent**: drafts that flipped from `status: draft` to `status: sent`.
- **Meetings completed**: prep files where the meeting date has passed.

### 3. Plan vs reality

If a plan-file exists for the week:
- Mark each planned action as `done` | `partial` | `not-started` | `obsolete`.
- Compute completion rate (raw — don't editorialize).

### 4. Reflective Q&A (skip if `--quick`)

Ask the user, one at a time:
1. What's one thing that worked this week?
2. What's one thing that didn't?
3. What's one thing you'd carry into next week?
4. Anything that should be added to ground-truth based on what you learned?

Capture answers verbatim.

### 5. Write the log

```markdown
# Week of {{week-of}} — log

> Generated: {{ISO timestamp}}
> Plan: {{plan-file or "no plan was made"}}
> Previous log: {{prev-log or "first week logged"}}

## Plan vs reality

| action | status | note |

> Completion: {{n}}/{{total}} actions

## Outreach diff

- New rows: ...
- Status changes: ...

## Opportunities diff

- Stage changes: ...

## Artifacts produced

- Briefs: ...
- Recommendations runs: ...
- Drafts sent: ...
- Meetings completed: ...

## Reflection (if not --quick)

**What worked**: ...

**What didn't**: ...

**Carry into next week**: ...

**Ground-truth deltas surfaced**: ...

## Suggested seeds for next week's plan

- ...
- ...
```

### 6. If ground-truth deltas surfaced

Don't write to ground-truth. Surface the proposed deltas with a recommendation: `/career:ground-truth edit-section <section>`.

### 7. Print summary

```
log: plans/<date>-week-log.md
plan completion: <n>/<total>
outreach activity: <n> rows changed
artifacts: <n>

next: /career:plan week (uses this log's "suggested seeds" + state diff)
```

## Guardrails

- Don't editorialize the diff. Numbers + facts; the reflection section is where interpretation lives.
- Plan-vs-reality is honest. "Completed 2/6" is more useful than "made progress on most".
- Reflection questions are open-ended; capture verbatim. Don't paraphrase the user.

## Failure modes

- **No plan-file for the week** → still produce diff + reflection, mark plan section as `## No plan was made for this week`.
- **First week ever** → no diff possible; produce a "starting state" snapshot instead.

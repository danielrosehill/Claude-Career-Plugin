---
description: Reconciliation pass — surface drift between ground-truth.md, CRM rows, and recent activity. Every finding cites a specific source row. Never silently overwrites ground-truth — proposes edits the user reviews.
---

# /career:reconcile

Single-skill command. Runs the `self-healing-context` skill.

## Usage

```
/career:reconcile [--strict] [--apply]
```

## Options

- `--strict` — surface even minor signal mismatches.
- `--apply` — for mechanical findings only (e.g. flip stale outreach row to `stalled`), confirm + apply per-item. Never applies to ground-truth.

## What it checks

- **Critical** — internal contradictions (ground-truth says X currently true; evidence shows X is false).
- **Drift** — direction shifts (ground-truth says employment, recent outreach is consulting-heavy).
- **Staleness** — rows untouched beyond reasonable thresholds.
- **Coverage gaps** — ground-truth domains with no domain-notes; companions configured but not installed.

## Output

`${WORKING_FOLDER}/reconcile/<YYYY-MM-DD>-findings.md`. Each finding has:
- Evidence (cited source).
- Rule that fired.
- Proposed action.
- Exact next command.

## Cadence

Run weekly (Friday or before `/career:plan week`) or whenever something feels off. Cheap to run, free to ignore.

## Notes

- Ground-truth edits are *always* proposals. The user owns those changes via `/career:ground-truth edit-section`.
- Mechanical staleness updates can be `--apply`'d after per-row confirmation.
- Empty workspace bails gracefully — reconciliation needs ground-truth + at least some CRM activity to be useful.

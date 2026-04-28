---
name: self-healing-context
description: Reconciliation pass. Reads ground-truth.md + crm/companies.md + crm/opportunities.md + crm/outreach.md and surfaces discrepancies. Never silently overwrites — every finding is a proposed delta the user reviews. Writes to <WORKING_FOLDER>/reconcile/<YYYY-MM-DD>-findings.md.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Self-Healing Context

Workspace state drifts: ground-truth says "currently working with X" but X's engagement closed two months ago; ground-truth says "looking for full-time" but recent outreach is 90% consulting. This skill surfaces the drift; the user owns the reconciliation.

## Inputs

`$ARGUMENTS`:
- Optional: `--strict` — surface even minor signal mismatches.
- Optional: `--apply` — for findings the user pre-approves, apply changes (rare; default is propose-only).

### Examples

```
/career:reconcile
/career:reconcile --strict
```

## Procedure

### 1. Load state

- `ground-truth.md` (all sections).
- `crm/companies.md`, `crm/opportunities.md`, `crm/outreach.md`.
- Recent week-logs (last 4 weeks) from `plans/`.
- Recent recommendations (last 30 days).

### 2. Run discrepancy checks

Each check has a description, the rule, and the action.

#### a) Engagement state vs ground-truth

- ground-truth `looking-for` says "currently working with client X" but `crm/opportunities.md` shows that engagement at status `closed-won` 90+ days ago, or `closed-lost`, or `paused` → propose ground-truth update.
- ground-truth lists "currently working with X" but no row for X exists in opportunities → propose adding the row or removing the claim.

#### b) Direction vs activity

- ground-truth says "looking for full-time" but ≥75% of last 30 days outreach used consulting templates → flag drift.
- ground-truth says "freelance only" but recent outreach mentions employment-shaped roles → flag drift.

#### c) Constraints vs targets

- ground-truth `hard-constraints` excludes a region (e.g. US-only) but recent recommendations include US-only companies → flag the recommendation logic, not ground-truth.
- ground-truth salary floor exceeds the highest known band on companies in `recommendations/` → flag mismatch.

#### d) Stale CRM rows

- `crm/companies.md` rows with status `engaged` but no outreach.md activity in 60+ days → propose status change to `paused` or `passed`.
- `crm/outreach.md` rows in status `sent` for >45 days with no follow-up → propose status `stalled`.

#### e) Domain coverage

- Recent week-logs show outreach concentrated in 1 domain while ground-truth `domains-of-interest` lists 4+ → flag narrowness.
- ground-truth `domains-of-interest` lists a domain with no domain-notes file → suggest running `/career:discover by-domain` for it.

#### f) Companion configuration

- `companions.transcription` set in config but companion not actually installed → flag.
- Outreach drafts using business-brand exist but `freelance-brand` in ground-truth is empty → flag.

### 3. Severity

Tag each finding:
- `critical` — internal contradiction (claim X is currently true, evidence X is false).
- `drift` — direction has shifted; ground-truth probably needs editing.
- `staleness` — row hasn't been touched, low cost to fix.
- `coverage` — gap (a domain not yet researched, a config not set).

`--strict` surfaces signal mismatches that would otherwise be filtered (small drifts, single-row stale rows).

### 4. Write the findings file

```markdown
# Reconcile findings — {{date}}

> Generated: {{ISO timestamp}}
> Sources: ground-truth.md ({{mtime}}), crm/* ({{summary}}), recent recommendations + week-logs.

## Critical

### {{finding-id}}: {{title}}
- evidence: {{specific row + file}}
- rule: {{which check fired}}
- proposed action: {{e.g. "edit ground-truth section `looking-for`"}}
- to apply: `{{exact next command}}`

## Drift

(same shape)

## Staleness

(same shape)

## Coverage gaps

(same shape)

## No discrepancies found

(if a category is clean, list it here so the user sees the system checked)
```

### 5. Optionally apply

If `--apply` and a finding has a clearly-mechanical action (e.g. flip outreach row from `sent` → `stalled` after 45 days):
- Show diff.
- Ask per-finding: y/n.
- Apply only confirmed.

Never auto-apply ground-truth edits — those always remain proposals.

### 6. Print summary

```
findings: reconcile/<date>-findings.md
critical: <n>
drift: <n>
staleness: <n>
coverage: <n>

next:
  critical → /career:ground-truth edit-section <section>
  staleness → /career:reconcile --apply (mechanical updates only)
  coverage → /career:discover by-domain "<missing>"
```

## Guardrails

- **Never silently overwrite ground-truth.** Critical and drift findings are *always* proposals.
- Every finding cites a specific file + row. If you can't cite a source, the finding is unfounded — drop it.
- Don't moralize. "You haven't followed up with X in 50 days" is staleness, not a failure.
- Severity is conservative. Don't escalate `staleness` to `critical` because the user "should" act on it.

## Failure modes

- **Empty workspace** (no CRM, no recommendations) → return "nothing to reconcile yet — run /career:discover and /career:research-company first".
- **Ground-truth missing** → bail; reconciliation needs ground-truth as the anchor.

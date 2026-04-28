---
name: skill-gap-analysis
description: Diff current skill inventory against target-role/domain profile. Categorises gaps as critical/material/met. Carries plan notes forward across refreshes. Writes skills/gaps.md. Different shape from suggest-skills (which is forward-looking against a horizon) — this is the inventory delta.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(test *), Bash(date *)
---

# Skill Gap Analysis

The inventory delta. `suggest-skills` proposes what to learn given a horizon and a domain; `skill-gap-analysis` measures what's missing right now against a stated target profile.

Run after editing inventory or target profile. Refreshes preserve plan notes.

## Inputs

`$ARGUMENTS`:
- Optional: `--target=<file-path>` — alternate target profile. Default: `${WORKING_FOLDER}/skills/target-profile.md`.
- Optional: `--inventory=<file-path>` — alternate inventory. Default: `${WORKING_FOLDER}/skills/inventory.md`.

## Prerequisites

- `skills/inventory.md` — user's current skills, each with a self-rated level (1–5 or `have/partial/gap`).
- `skills/target-profile.md` — required skills with required level.

If either is missing, bail and instruct the user to create them (or run `/career:onboard`).

## Procedure

### 1. Read both files

Parse rows. Tolerate either numeric (1–5) or named levels — normalise internally to 0–5 (have=5, partial=3, gap=0).

### 2. Match

For each skill in target-profile:
- Find matching row in inventory (exact slug, then fuzzy on label). If ambiguous (≥2 candidates within edit-distance threshold), ask the user.
- If no match, treat current = 0.

### 3. Categorise

`gap = required - current`:
- `gap >= 2` → **critical**
- `gap == 1` → **material**
- `gap <= 0` → **met**

### 4. Carry forward plans

If a previous `skills/gaps.md` exists, parse its rows. For each skill still in the new gaps list, copy the **plan** column forward.

For *new* critical/material gaps, leave plan blank and surface them in the summary.

### 5. Write gaps.md

```markdown
# Skill gaps

**Last refreshed:** {{date}}
**Target:** {{target-profile path}}
**Inventory:** {{inventory path}}

## Critical

| skill | required | current | gap | plan |
| --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... |

## Material

(same shape)

## Met

(skill | required | current — minimal columns)

## Notes

{{summary of what changed since last refresh}}
```

### 6. Print summary

```
gaps refreshed: skills/gaps.md
critical: <n>  material: <n>  met: <n>
new since last refresh: <list>
no plan yet: <list>

next:
  critical → /career:suggest projects (close a gap with a shippable artifact)
            or /career:suggest skills (learning plan within horizon)
```

## Guardrails

- Don't drop met skills silently — they're proof of progress and useful for the user.
- If inventory and target are both empty, bail with a setup pointer.
- Carry-forward is conservative: if the underlying skill row was renamed in either file, ask before transferring the plan.

## Failure modes

- Inventory or target missing → bail.
- Plan carry-forward ambiguous → ask per row.
- No gaps at all → still write gaps.md (with "Met" only) and celebrate briefly.

---
name: ground-truth
description: Read or edit the workspace's ground-truth.md — the source-of-truth profile every Career-OS skill reads first. Use when the user says "update my profile", "I changed roles", "I'm looking for X now", or after a major life/career change. Idempotent and diff-friendly. Never silently overwrites.
disable-model-invocation: false
allowed-tools: Read, Edit, Write, Bash(test *), Bash(cat *)
---

# Ground Truth — read/edit

`ground-truth.md` lives at the root of the user's `WORKING_FOLDER` (read from `~/.config/career-os/config.json`). It's the single source of truth that every other skill reads before doing anything. Stale or missing ground-truth → bad recommendations.

## When to invoke

- User says "update my profile / ground-truth / who-I-am".
- After `/career:onboard` (called as the final foundational step).
- User reports a status change: new role, ended engagement, new constraint, new domain of interest, new company contacted.
- `/career:reconcile` flagged a discrepancy and the user accepted the proposed update.

## Sections (per spec §4.1)

The doc has these named sections — preserve order, preserve any sections not in this list:

1. **Who I am** — 2–4 sentences.
2. **Resume pointer** — relative path, defaults to `resume/master.md`.
3. **Looking for** — `employment | contract | both` + freeform detail.
4. **Salary expectations** — currency, floor, target, notes.
5. **Hard constraints** — location, hours, family/life, other.
6. **Domains of interest** — bulleted list of niches.
7. **Companies already engaged** — quick-reference table; full log is `crm/outreach.md`.
8. **Branding presence** — website / GitHub / LinkedIn / HF / other.
9. **Freelance brand** — only if `Looking for` includes contract.

## Procedure

### 1. Resolve the path

Read `~/.config/career-os/config.json` for `WORKING_FOLDER`. If config is missing, prompt the user to run `/career:onboard` first.

Target file: `${WORKING_FOLDER}/ground-truth.md`.

### 2. Decide mode

Three modes:

- **read** — user wants to see current state. Cat the file, render section-by-section.
- **edit-section** — user wants to change one section. Show current value, ask for new value, write back, confirm diff.
- **bulk-edit** — user wants to walk through every section. Ask per section; skip with "no change" / "same".

If unclear from the user's message, default to **read** and offer the other two.

### 3. Edit safely

- Always read the current file first.
- Use Edit with exact-string replacement on the target section's content (keep the section header intact).
- After every edit, show a unified diff and ask the user to confirm before saving the next change.
- Never use replace_all unless the user explicitly opts in.
- Never delete a section — if the user wants it gone, blank the body and leave the heading with `_(none)_`.

### 4. Cascade notifications

After any edit that affects:

- **Looking for** → tell the user that `suggest-companies` / `suggest-roles` will rerank next run.
- **Hard constraints (location, remote)** → tell the user `discover-remote-friendly` cache may be stale; offer to clear `cache/`.
- **Salary expectations** → mention that salary-research cache is keyed by (geo, role); editing here doesn't auto-invalidate.
- **Companies already engaged** → remind that `crm/outreach.md` is the canonical log; ground-truth is a quick-reference only.

### 5. Write minimal commit-style summary

After save, print one-line summary: `Updated: <section>` per change. No prose.

## Idempotency

- Running ground-truth on an unchanged file is a no-op.
- Running with the same input twice produces the same file.
- Re-running after partial edits (e.g. user Ctrl-C'd mid-flow) picks up where it left off — never duplicates section headings.

## Failure modes

- **Config missing** → prompt to run `/career:onboard`. Don't proceed.
- **`ground-truth.md` missing** → offer to create from template (`template/career-os/ground-truth.md` in the plugin repo). Confirm before writing.
- **`WORKING_FOLDER` path doesn't exist** → bail with a clear error; do not auto-create (the workspace should already exist via `new-workspace`).

## Output discipline

- No emojis.
- No "I've updated..." narration. The diff is the narration.
- One-line summaries per change.

---
name: brief-cultural-fit
description: Produce a Cultural Fit research brief — pairs the user's stated operating style and constraints (read from ground-truth.md) against the company's apparent operating mode (from public signals). Output is a match-map plus blunt friction list. Use after the other briefs are in hand. Requires populated ground-truth.md. Reads templates/research-briefs/cultural-fit.md and writes to <WORKING_FOLDER>/research/companies/<slug>/cultural-fit.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Brief — Cultural Fit

The brief that pairs user's operating style with the company's. Most consequential for retention, least consequential for getting an interview. Run it last — it benefits from the other briefs already being on disk.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company_name>`.
- Optional: `--slug=<slug>`.
- Optional: `--lens=<employer|client|partner>` (default `employer`).

## Procedure

### 1. Resolve config + paths

`~/.config/career-os/config.json` → `WORKING_FOLDER`. Output: `${WORKING_FOLDER}/research/companies/${slug}/cultural-fit.md`.

### 2. Read inputs

- Template: `${PLUGIN_ROOT}/templates/research-briefs/cultural-fit.md`.
- **Required**: `${WORKING_FOLDER}/ground-truth.md`. If missing or essentially empty, prompt the user to run `/career:ground-truth` first; do not proceed with placeholder values.
- If they exist, also read other briefs in the same dir (`company-overview.md`, `glassdoor-signal.md`, `recruitment-profile.md`) for input.

### 3. Pull user's stated style

From `ground-truth.md`:
- Looking for (employment / contract / both).
- Hard constraints (location, hours, family, other).
- Domains of interest.

If ground-truth doesn't cover collaboration preferences / decision-making style / what drains vs energises: ask the user inline. Three short questions. Don't fabricate.

### 4. Read company's operating mode

From public signals — engineering blog posts, podcast appearances, public Slack/Discord activity, careers-page values, Glassdoor culture themes, public memos. Cite each.

Categorise:
- Communication mode (written-first / meeting-first / sync-heavy / async-first).
- Decision velocity.
- Hierarchy.
- Operating cadence.
- On-call expectations.
- Documented values + whether they appear operationalised.

If `glassdoor-signal.md` exists in the workspace, read its theme analysis as a primary input. Don't re-research what's already on disk.

### 5. Match map

Per-dimension table comparing user vs company. Fit values: `match | tension | mismatch | unknown`.

### 6. Outright frictions

Hard mismatches with concrete examples. Be blunt.

### 7. Soft frictions / things to probe

Open questions that would resolve `uncertain` verdicts. Frame as questions the user would ask in an interview / discovery call.

### 8. Verdict

- **Cultural fit**: `strong | likely | uncertain | poor`
- **Confidence**: `high | medium | low`
- **Headline reason**: one sentence.

Decision rules:
- `strong` requires at least one specific evidence pair (user dimension → company signal).
- `poor` is valuable; do not soften.
- `uncertain` is acceptable when ground-truth is sparse or company signals are weak.

### 9. Write output + update CRM

Append `+cultural` to the row.

## Companion delegation

If `decision-evaluation-framework` is installed, the brief output is consumable by `decision-evaluation-framework:swot` if the user wants a structured pivot/decline analysis later.

## Guardrails

- Ground-truth is privileged — don't override with company marketing.
- "Strong fit" requires evidence pairs. Generic praise is not allowed.
- "Poor fit" is not softened.
- Run last among the six briefs; benefits from the other outputs.

## Failure modes

- **`ground-truth.md` missing or empty** → bail; prompt to run `/career:ground-truth`.
- **No public signals available (stealth company)** → write `## Status: low-signal — need to ask directly`; produce the questions section without a verdict.

## Idempotency

- Re-run prompts before overwrite.
- `--refresh` silently overwrites.

---
name: companies-like-x
description: Given one seed company, find peers along an explicit similarity axis (product, customer, tech stack, business model, culture). Writes to <WORKING_FOLDER>/domain-notes/like-<seed-slug>.md. Reads outreach.md to mark contact status. Use when the user has a known reference point and wants more of the same shape.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Companies Like X

Given a seed company, enumerate peers along a chosen similarity axis. Avoids generic "competitors of X" — forces the user to pick the dimension.

## Inputs

`$ARGUMENTS`:
- **Required**: `<seed>` — company name or domain.
- Optional: `--axis=<axis>` — one of `product`, `customer`, `stack`, `business-model`, `culture`. Default: ask.
- Optional: `--n=<count>` (default 15).
- Optional: `--exclude-direct-competitors` — skip head-to-head rivals; surface adjacent peers instead.
- Optional: `--refresh`.

### Examples

```
/career:discover companies-like-x snowglobe.ai --axis=product
/career:discover companies-like-x linear --axis=culture --n=20
/career:discover companies-like-x retool --axis=customer --exclude-direct-competitors
```

## Procedure

### 1. Resolve config + paths

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. Output: `${WORKING_FOLDER}/domain-notes/like-${seed-slug}.md`.

### 2. Profile the seed

Single-pass research on `<seed>`:
- One-line product description.
- Customer segment (ICP).
- Tech stack signals (job postings, engineering blog).
- Business model (b2b saas, marketplace, prosumer, open-core, services).
- Culture signals (remote posture, values, comp band, public hiring tone).

If `--axis` not provided, surface the five axes with a one-liner each and ask the user to pick before continuing.

### 3. Enumerate peers along the axis

WebSearch + WebFetch around the chosen axis only. Examples:

- `product`: companies solving the same job for the user.
- `customer`: different products serving the same buyer.
- `stack`: companies using the same core tech (e.g., "Rust + Postgres + b2b API").
- `business-model`: open-core peers, prosumer-saas peers, etc.
- `culture`: remote-first, async-default, transparent comp, etc.

Aim for `--n` matches. Drop matches that only fit on a different axis (note them in "Out of axis").

### 4. Cross-reference outreach.md

Same as `discover-by-domain` step 4. Mark each peer with `contact-status`.

### 5. Write the note

Format:

```markdown
# Companies like {{seed}} — axis: {{axis}}

> Seed: {{seed}}
> Axis: {{axis}}
> Surveyed: {{date}}
> Peers enumerated: {{n}}

## Seed profile

- Product: ...
- Customer: ...
- Stack: ...
- Business model: ...
- Culture: ...

## Why this axis

{{1–2 sentences on what "similar on {{axis}}" means here}}

## Peers

| name | website | why-similar (one line) | stage | hq | remote? | contact-status | notes |
| --- | --- | --- | --- | --- | --- | --- | --- |

## Out of axis (similar on a different dimension)

- {{name}} — similar on {{other-axis}}, not {{chosen-axis}}.

## Sources

1. ...
```

### 6. Print summary

```
peers note: domain-notes/like-<seed-slug>.md
axis: <axis>
peers: <n>
never-contacted: <n>
contacted-stale: <n>
contacted-recent: <n>

next: /career:suggest companies --from=like-<seed-slug>
```

## Guardrails

- One axis per run. Re-run with a different `--axis` for a different lens — don't blur axes.
- "Why-similar" must be specific to the axis, not generic ("also a SaaS company" is not a reason).
- No fabrication. If you can't find evidence for the claimed similarity, drop the row.

## Failure modes

- **Seed unknown / no signal** → bail; ask the user for a website or a clarifying note.
- **Axis ambiguous** ("similar but better") → ask the user to pick from the five axes.

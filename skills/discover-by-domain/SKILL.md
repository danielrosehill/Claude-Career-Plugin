---
name: discover-by-domain
description: Enumerate the main companies in a defined niche (e.g. "agentic AI for policy simulation", "open-source LLM tooling"). Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>.md. Reads outreach.md to flag already-contacted companies. Use as the first step before /career:suggest companies — populate the domain knowledge once, suggest from it many times.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Discover by Domain

Given a niche, enumerate the main companies operating in it. Builds a per-domain markdown note that downstream skills read from.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>` — free text describing the niche. First positional, often quoted.
- Optional: `--slug=<slug>` (default: kebab-case of domain).
- Optional: `--n=<count>` (default 20) — target company count.
- Optional: `--include-stealth` — include companies in stealth where signals exist (founders' prior backgrounds, hires).
- Optional: `--refresh` — overwrite existing domain note.

### Examples

```
/career:discover by-domain "agentic AI for policy simulation"
/career:discover by-domain "open-source LLM tooling" --n=30
/career:discover by-domain "climate-tech instrumentation" --include-stealth
```

## Procedure

### 1. Resolve config + paths

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`. Output path: `${WORKING_FOLDER}/domain-notes/${slug}.md`. If exists and no `--refresh`, ask whether to extend, replace, or open existing.

### 2. Define the niche

If `<domain>` is broad ("AI", "fintech"), ask the user to narrow before proceeding — broad domains belong in `chunked-discovery`, not here.

Capture (for the note's preamble):
- Definition: 1–2 sentences.
- What's in scope.
- What's out of scope (adjacent niches that get conflated).

### 3. Enumerate

WebSearch + WebFetch. Sources to cover:
- Industry "landscape" articles / VC theses / market maps.
- Funding announcements in the last 24 months in the niche.
- Conference speaker lists / sponsor lists for relevant events.
- LinkedIn companies-by-industry filters.
- HN / Reddit threads with "show me companies doing X".

Aim for `--n` companies. Stop earlier if the long tail becomes off-topic.

### 4. Cross-reference outreach log

Read `${WORKING_FOLDER}/crm/outreach.md`. For each enumerated company, mark:
- `contacted-recent` — outreach within last 90 days.
- `contacted-stale` — outreach >90 days ago.
- `never-contacted` — no row.

This is *informational* in this skill — `suggest-companies` is where the anti-loop hard logic lives. Surface here so the user sees coverage at a glance.

### 5. Write the note

Format:

```markdown
# Domain — {{domain_name}}

> Slug: {{slug}}
> Surveyed: {{date}}
> Companies enumerated: {{n}}

## Definition

{{1–2 sentences}}

## In scope

- ...

## Out of scope

- ... (related but not this niche)

## Companies

| name | website | stage | hq | remote? | contact-status | notes |
| --- | --- | --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... | never-contacted / contacted-recent / contacted-stale | ... |

## Recent funding (last 24 months)

- ...

## Adjacent niches to consider next

- ...

## Sources

1. ...
```

### 6. Print summary

```
domain note: domain-notes/<slug>.md
companies enumerated: <n>
never-contacted: <n>
contacted-stale: <n>
contacted-recent: <n>

next: /career:suggest companies --domain=<slug>
```

## Guardrails

- Don't pad. If the niche only has 7 serious companies, the note has 7 companies. `--n` is a target ceiling, not a quota.
- "Stage" column means funding stage where known, `unknown` otherwise. No fabrication.
- Mark inferred fields explicitly.

## Failure modes

- **Domain too broad** → bail with `chunked-discovery` recommendation.
- **Web fetch failures** → write what you have; mark note as `## Status: partial`.

## Idempotency

- Re-run without `--refresh` → asks before overwrite; `extend` mode appends new companies, leaves existing rows alone.
- `--refresh` overwrites silently.

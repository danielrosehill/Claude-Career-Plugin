---
name: suggest-projects
description: Suggest concrete projects (build / write / contribute) the user could ship to close a skill gap, demonstrate a capability, or create a hook for outreach to a target company. Writes to <WORKING_FOLDER>/recommendations/projects-<date>.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Suggest Projects

Translates "what should I work on" into concrete shipped artifacts. Each project has a clear scope, time estimate, and what it would unlock (which gap it closes, which conversation it starts).

## Inputs

`$ARGUMENTS`:
- Optional: `--close-gap=<skill>` — close a specific skill gap.
- Optional: `--target-company=<slug>` — produce a project that's a credible hook for outreach to this company.
- Optional: `--target-domain=<slug>` — domain-shaped project (signal-building).
- Optional: `--time-budget=<weekend|week|month>` (default `week`).
- Optional: `--n=<count>` (default 5).

### Examples

```
/career:suggest projects --close-gap="RLHF tooling" --time-budget=weekend
/career:suggest projects --target-company=snowglobe --time-budget=week
/career:suggest projects --target-domain=ai-evals --n=8
```

## Procedure

### 1. Load context

- `ground-truth.md`.
- Source signals: skills note, company briefs, domain note.

### 2. Generate candidates

For each candidate project, define:
- **Shape**: build / write / contribute / present.
- **Concrete artifact**: repo / blog post / OSS PR / talk / dataset / template.
- **Time estimate**: maps to `--time-budget`.
- **What it demonstrates**: ties back to a specific skill or domain claim.
- **What it unlocks**: gap closed, hook for outreach, portfolio piece.
- **Distribution**: where the artifact lives + first 3 places to share it.

### 3. Filter

- Drop anything that needs more than 2× the time budget.
- Drop anything that requires private/proprietary access the user doesn't have.
- Keep diversity: don't suggest 5 blog posts; mix shapes.

### 4. Write the recommendations file

```markdown
# Project suggestions — {{date}}

> Goal: {{close-gap | target-company | target-domain}}
> Time budget: {{budget}}
> Generated: {{datetime}}

## Projects

### 1. {{title}}
- shape: {{build|write|contribute|present}}
- artifact: {{repo|post|PR|talk|dataset}}
- time: {{est}}
- demonstrates: {{skill / claim}}
- unlocks: {{gap closed | outreach hook | portfolio}}
- distribution: {{where to publish + share}}
- starter steps:
  1. ...
  2. ...

### 2. ...

## Why these and not others

- {{1–2 sentences on the trade-offs you made}}

## Sources

1. ...
```

### 5. Print summary

```
recommendations: recommendations/projects-<date>.md
projects: <n>

next: pick one and commit to a time slot in /career:plan
```

## Guardrails

- A project must be shippable in the time budget. "Write a paper on X" without scoping is not a project.
- Hook-shaped projects must have a real connection to the target. Don't manufacture a fake project to "have something to send".
- Don't suggest closed-source replicas of the target's product as a hook — it reads as competitive, not collaborative.

## Failure modes

- **Time budget < gap closing effort** → recommend a smaller scope or extend budget.
- **No target signal** → ask for `--close-gap` or `--target-domain`.

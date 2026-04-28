---
name: suggest-consulting-packages
description: For users with a freelance-brand in ground-truth, propose narrow consulting offers (problem framed, scope outlined, rough investment indicated) targeted at a domain or company. Writes to <WORKING_FOLDER>/recommendations/consulting-packages-<date>.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Suggest Consulting Packages

Translates the freelance-brand into 3–5 narrow offers. Each package is small enough to pitch cold, specific enough to be credible, and shaped so the buyer can say yes without committee.

## Inputs

`$ARGUMENTS`:
- Optional: `--domain=<slug>`.
- Optional: `--target-company=<slug>`.
- Optional: `--n=<count>` (default 5).
- Optional: `--shape=<audit|build|advisory|teach>` — bias toward one shape.

At least one of `--domain` / `--target-company` recommended.

### Examples

```
/career:suggest consulting-packages --domain=ai-evals
/career:suggest consulting-packages --target-company=snowglobe --shape=audit
/career:suggest consulting-packages --domain=devtools --shape=build --n=3
```

## Procedure

### 1. Load context

- `ground-truth.md` (especially `freelance-brand` and `branding`).
- Optional briefs / domain notes.

If `freelance-brand` is empty, bail with: "no freelance-brand in ground-truth — run /career:ground-truth edit and add it before consulting packages will be useful."

### 2. Generate candidate packages

Each package has:
- **Title** (short, specific): "AI evals audit for series-A devtools".
- **Problem framed** (one sentence): the pain the buyer recognizes.
- **Outcome** (one sentence): what they have at the end.
- **Scope outline**: 3–6 bullet deliverables.
- **Time / shape**: e.g. "2 weeks, async, 1 sync per week".
- **Rough investment**: a band, not a number ("low-mid 5 figures") or omit if user prefers — mark `[ask user]`.
- **Buyer**: title of the person who would sign off.
- **Disqualifiers**: who this is *not* for (so cold pitches don't waste both sides' time).

### 3. Pressure-test

For each package, check:
- Is it narrow enough to fit in a 130–200 word cold pitch? If not, narrow.
- Does it depend on access the buyer wouldn't grant a stranger? If so, redesign or stage.
- Is the outcome inspectable? ("They have an artifact / decision / install".) If not, sharpen.

### 4. Write the recommendations file

```markdown
# Consulting package suggestions — {{date}}

> Scope: {{domain | company}}
> Shape bias: {{audit|build|advisory|teach|none}}
> Generated: {{datetime}}

## Packages

### 1. {{title}}
- problem: ...
- outcome: ...
- scope:
  - ...
- shape: {{time + working pattern}}
- investment: {{band or [ask user]}}
- buyer: {{title}}
- not for: {{disqualifier}}

### 2. ...

## How to use these in outreach

- Pair with template `consulting-pitch` via `/career:outreach draft <slug> --template=consulting-pitch`.
- The pitch should pick *one* package and offer to discuss; never list multiple in one email.

## Sources

1. ...
```

### 5. Print summary

```
recommendations: recommendations/consulting-packages-<date>.md
packages: <n>

next: /career:outreach draft <company> --template=consulting-pitch --brief-as=business
```

## Guardrails

- One package per pitch. If you can't pick one, the offer is not narrow enough.
- Don't promise outcomes that depend on the buyer's existing data quality / org alignment without saying so.
- "Investment" bands should be honest. If the user has not set pricing, leave `[ask user]` rather than fabricating.

## Failure modes

- **No freelance-brand in ground-truth** → bail with instruction to populate it.
- **Domain too broad** → ask for narrower scope or a specific company target.

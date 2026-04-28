---
name: map-investors
description: Map VCs and angels active in a domain. Output is usable as introducer surface — for each investor, surface portfolio companies that overlap with the user's target list. Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>/investors.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Map Investors

For each investor, list their relevant portfolio companies + ones that overlap with the user's outreach targets. The point isn't to pitch the VC — it's to find the people who could introduce the user to the founders they want to meet.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>`.
- Optional: `--stage=<seed|series-a|growth|all>` (default `seed,series-a`).
- Optional: `--geography=<region>` — defaults `USER_LOCATION` + global.
- Optional: `--n=<count>` (default 15).
- Optional: `--cross-ref=<domain-slug>` — overlay portfolio against another domain note.

### Examples

```
/career:ecosystem investors "ai evals"
/career:ecosystem investors "devtools" --stage=seed --geography="EMEA"
/career:ecosystem investors "ai agents" --cross-ref=ai-eval-startups
```

## Procedure

### 1. Paths + load

Read `crm/companies.md` (for target companies). If `--cross-ref`, read that domain note's company list.

Output: `${WORKING_FOLDER}/domain-notes/${domain-slug}/investors.md`.

### 2. Enumerate

For each investor / fund, capture:
- Name (firm + lead partner if known).
- Stage focus.
- Geography focus.
- Domain thesis (1 line, cited).
- Recent investments in the domain (last 24 months) — names + dates.
- Portfolio overlap with user's target / cross-ref list — this is the high-signal field.
- Public outreach surface: do they have a public submission form, an investor email, conferences they regularly attend.
- Is the lead partner active on a public surface (substack, twitter, podcast).

### 3. Write the note

```markdown
# Investors — {{domain}}

> Domain: {{domain}}
> Stage filter: {{stage}}
> Geography: {{geography}}
> Cross-ref: {{cross-ref or none}}
> Surveyed: {{date}}

## High overlap with target list

| investor | lead partner | stage | recent {{domain}} investments | overlap with my list | outreach surface | source |

## Domain-active, no overlap (background)

(same columns)

## Inactive in this domain (for the record)

| investor | last domain investment |

## Sources

1. ...
```

### 4. Print summary

```
note: domain-notes/<domain-slug>/investors.md
high-overlap: <n>
domain-active: <n>

next: 
  high-overlap → introducer surface — note who in your network connects to these partners
  follow-up → /career:outreach draft (light, soft-touch templates only)
```

## Guardrails

- This skill maps; it doesn't recommend pitching. The output is *introducer surface*, not a target list.
- Portfolio overlap claims must be backed by Crunchbase / public announcements / firm portfolio pages. Cite sources.
- Don't surface stealth bets that require non-public knowledge.

## Failure modes

- **Domain has no clear investor cluster** → write the note as `## Status: niche too early/quiet for clear VC mapping` and surface adjacent investors.

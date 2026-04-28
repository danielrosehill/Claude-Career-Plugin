---
name: find-communities
description: Enumerate communities (LinkedIn groups, Slack channels, Discords, professional orgs, mailing lists) for a domain. Each entry includes activity-level signal and alignment to ground-truth. Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>/communities.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Find Communities

Where the conversation actually happens for this niche. Filtered by activity (dead Slacks are noise) and alignment (some communities won't take the user's profile).

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>`.
- Optional: `--types=<slack,discord,linkedin,reddit,mailing-list,org>` (default: all).
- Optional: `--n=<count>` (default 15).
- Optional: `--gated-ok` — include invite-only / paid communities.

### Examples

```
/career:ecosystem communities "ai evals"
/career:ecosystem communities "devtools" --types=slack,discord
/career:ecosystem communities "policy tech" --gated-ok
```

## Procedure

### 1. Paths

Output: `${WORKING_FOLDER}/domain-notes/${domain-slug}/communities.md`.

### 2. Enumerate

WebSearch + WebFetch. For each candidate community, capture:
- Name + platform.
- Join URL (or "request access via X").
- Activity signal: rough message volume / week, last visible post if scrapeable.
- Membership shape: practitioner / academic / vendor / mixed.
- Cost: free / paid / invite-only.
- Alignment score against ground-truth (0–3): does the user fit; is this a buyer crowd, peer crowd, or learning crowd.

### 3. Write the note

```markdown
# Communities — {{domain}}

> Domain: {{domain}}
> Types: {{types}}
> Surveyed: {{date}}

## Active + aligned (join these)

| name | platform | join | activity | shape | cost | alignment | source |

## Active but mixed-fit (lurk first)

(same columns)

## Dormant / low-signal (note for the record)

| name | last activity |

## Sources

1. ...
```

### 4. Print summary

```
note: domain-notes/<domain-slug>/communities.md
active+aligned: <n>
mixed-fit: <n>
dormant: <n>

next: pick 1–2 active+aligned to actually join this week
```

## Guardrails

- Activity signal is mandatory. A community with no visible recent activity is dormant — don't recommend joining.
- Don't recommend more than 3 communities to actively engage with. Lurking elsewhere is fine.
- Alignment is honest: "great community, wrong audience for you" is a valid downgrade.

## Failure modes

- **Domain too narrow** (no public communities exist) → return the empty note with adjacent communities listed under `## Adjacent options`.
- **All candidates require auth to assess activity** → mark each with `activity: unknown — gated` and surface the list anyway.

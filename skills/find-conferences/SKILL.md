---
name: find-conferences
description: Enumerate conferences and events at the intersection of a domain, geography, and date range. Reads ground-truth for goals/travel constraints. Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>/conferences.md. Surfaces CFP windows separately so the user can submit, not just attend.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Find Conferences

Domain ∩ geography ∩ date-range. Output groups events by attend / submit / sponsor-only and tags those with currently-open CFPs.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>` — niche.
- Optional: `--geography=<region>` — defaults to `USER_LOCATION` + global virtual events.
- Optional: `--horizon=<next-6mo|this-year|next-12mo>` (default `next-12mo`).
- Optional: `--n=<count>` (default 12).
- Optional: `--include-virtual` (default true).
- Optional: `--cfp-only` — only events with currently-open CFP submissions.

### Examples

```
/career:ecosystem conferences "agentic AI"
/career:ecosystem conferences "devtools" --geography="EMEA" --cfp-only
/career:ecosystem conferences "ai evals" --horizon=next-6mo
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`, `USER_LOCATION` from config. Read `ground-truth.md` for travel-constraint signals.

Output: `${WORKING_FOLDER}/domain-notes/${domain-slug}/conferences.md` (creates the per-domain subdirectory if needed).

### 2. Enumerate

WebSearch + WebFetch:
- Major industry conferences in the niche.
- Regional events.
- Specialist workshops / unconferences.
- Vendor summits in the niche.
- Academic conferences if domain is research-flavored.

Aim for `--n` events. Always include at least 2 free/virtual options unless filtered out.

### 3. Capture per event

For each event:
- Name
- Dates (or "TBA YYYY-Q?" if not yet announced)
- Location (city + country, or "virtual")
- CFP status: `open until <date>` | `closed for <year>` | `no public CFP`
- Ticket cost band
- Travel estimate from `USER_LOCATION` (rough; mark `[ask user]` if not derivable)
- Prior-year signal: who attended, who spoke (cite source)
- Why-relevant — tie to ground-truth domain or skill claim
- Mode tag: `attend` (worth being in the room) | `submit` (CFP open + good fit) | `sponsor-only` (vendor-heavy, low IC value)

### 4. Write the note

```markdown
# Conferences — {{domain}}

> Domain: {{domain}}
> Geography: {{geography}}
> Horizon: {{horizon}}
> Surveyed: {{date}}

## Open CFPs (submit)

| event | dates | CFP closes | location | why submit | source |

## Worth attending

| event | dates | location | ticket | travel from {{loc}} | why | source |

## Sponsor-only / vendor-heavy

| event | note |

## Honest unknowns (TBA / no public info)

| event | what would resolve this |

## Sources

1. ...
```

### 5. Print summary

```
note: domain-notes/<domain-slug>/conferences.md
attend: <n>
submit (CFP open): <n>
sponsor-only: <n>

next:
  - submit → /career:suggest projects --target-domain=<slug> (talk-shaped project)
  - attend → /career:plan week (block travel + cost)
```

## Guardrails

- CFP windows are time-sensitive. Always cite the deadline date and source URL.
- "Worth attending" must be backed by who-attended evidence, not marketing copy.
- Don't pad. If the niche has 4 real events, the note has 4 events.

## Failure modes

- **Domain too broad** → ask to narrow.
- **No upcoming events** → write the note empty with a `## Status: nothing scheduled in horizon` note.

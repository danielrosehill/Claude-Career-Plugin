---
description: Discovery entry point — enumerate companies in a domain, find peers of a seed, scope by remote-friendliness or geography, or chunk a broad topic. All discovery skills write to <WORKING_FOLDER>/domain-notes/ for downstream /career:suggest to consume.
---

# /career:discover

Five sub-skills, one entry point. The output of every subcommand is a markdown note in `domain-notes/` that `/career:suggest companies` reads.

## Subcommands

```
/career:discover by-domain        <domain> [--n=20] [--include-stealth] [--refresh]
/career:discover companies-like-x <seed>   [--axis=product|customer|stack|business-model|culture] [--n=15]
/career:discover remote-friendly  <domain> [--location=<override>] [--strict] [--lens=employer|client]
/career:discover by-geography     <geography> <domain> [--include-offices] [--lens=employer|client]
/career:discover chunked          <broad-domain> [--segments=6] [--depth=10] [--auto-drill]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `by-domain` | `discover-by-domain` |
| `companies-like-x` | `companies-like-x` |
| `remote-friendly` | `discover-remote-friendly` |
| `by-geography` | `discover-by-geography` |
| `chunked` | `chunked-discovery` |

## When to use which

- **Know the niche** → `by-domain`.
- **Have one company you like** → `companies-like-x` (pick an axis).
- **Need timezone fit** → `remote-friendly`.
- **Need physical locality** → `by-geography`.
- **Topic is too big** ("AI", "fintech") → `chunked`, then drill per segment.

## End-to-end example

```
/career:discover by-domain "agentic AI for policy simulation"
# writes domain-notes/agentic-ai-for-policy-simulation.md

/career:suggest companies --domain=agentic-ai-for-policy-simulation
# reads the domain note + crm/outreach.md, applies anti-loop, writes recommendations/companies-<date>-<source>.md

/career:research-company <top-pick-slug>
# six briefs + SUMMARY for the highest-ranked candidate
```

## Notes

- Discovery skills are read + write-domain-note only. None of them touch CRM rows or send anything.
- `crm/outreach.md` is read for the `contact-status` column — informational here; the hard anti-loop logic lives in `suggest-companies`.
- Re-running a discovery without `--refresh` asks before overwriting an existing note.

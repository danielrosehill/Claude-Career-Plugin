---
description: Ecosystem entry point — conferences, communities, job boards, hackathons, investors, LinkedIn import. Lower-stress, high-leverage exploration that maps the surface around a domain. All outputs land in <WORKING_FOLDER>/domain-notes/<slug>/.
---

# /career:ecosystem

Six sub-skills, one entry point. Where `/career:discover` enumerates *companies*, `/career:ecosystem` enumerates the *surface around them* — events, groups, boards, hackathons, investors, your own LinkedIn graph.

## Subcommands

```
/career:ecosystem conferences     <domain> [--geography=...] [--horizon=next-6mo|this-year|next-12mo] [--cfp-only]
/career:ecosystem communities     <domain> [--types=slack,discord,linkedin,reddit,...] [--gated-ok]
/career:ecosystem boards          <domain> [--location=...] [--remote-only] [--include-generic]
/career:ecosystem hackathons      <domain> [--geography=...] [--horizon=3mo|6mo|12mo] [--virtual-only|--in-person-only]
/career:ecosystem investors       <domain> [--stage=seed|series-a|growth|all] [--cross-ref=<domain-slug>]
/career:ecosystem linkedin-import <zip-path> [--cross-ref=<domain-slug>] [--surface-followups]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `conferences` | `find-conferences` |
| `communities` | `find-communities` |
| `boards` | `find-jobs-boards` |
| `hackathons` | `find-hackathons` |
| `investors` | `map-investors` |
| `linkedin-import` | `import-linkedin-history` |

## When to use which

- **Conferences** — to attend or to submit (CFP-driven).
- **Communities** — to lurk in or contribute to.
- **Boards** — for ongoing passive surfacing of postings.
- **Hackathons** — as outreach surface (founders/judges in the room) more than as coding contests.
- **Investors** — for *introducer surface*; the goal is meeting founders, not pitching VCs.
- **LinkedIn-import** — to parse your own export and surface latent follow-ups + cross-refs against domain notes.

## End-to-end example

```
/career:discover by-domain "ai evals"
/career:ecosystem conferences "ai evals" --cfp-only
# → opens CFPs become "talk-shaped projects" via /career:suggest projects
/career:ecosystem hackathons "ai evals" --horizon=3mo
/career:ecosystem investors "ai evals" --cross-ref=ai-evals
# investor portfolios overlap with target companies → introducer surface
/career:ecosystem linkedin-import ~/Downloads/li.zip --cross-ref=ai-evals --surface-followups
# already-connected at target companies? skip Hunter for those
```

## Output layout

All ecosystem outputs land per-domain:

```
domain-notes/
  ai-evals.md                    # discovery output
  ai-evals/
    conferences.md
    communities.md
    jobs-boards.md
    hackathons.md
    investors.md
linkedin/
  profile.md
  connections.csv
  messages/
  cross-ref-ai-evals.md
  follow-ups.md
```

## Notes

- Ecosystem skills are read + write-domain-note only. None of them touch CRM rows or send anything.
- LinkedIn import lands in `linkedin/`; ensure your workspace `.gitignore` excludes it if you don't want it committed.
- Cross-referencing investors against a domain's company list is the most leveraged use — surfaces who could introduce you to whom.

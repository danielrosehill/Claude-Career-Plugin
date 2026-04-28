---
description: White-hat salary benchmarking for a (geography, role) pair. Cached so repeat calls don't re-fetch.
---

# /career:salary-benchmark

Single-skill command. Runs the `salary-research` skill.

## Usage

```
/career:salary-benchmark --role=<title> --geo=<market> [--seniority=<...>] [--currency=<CCY>] [--company=<slug>] [--refresh]
```

## What it does

- White-hat sources only: postings, salary platforms, BLS/Eurostat, SEC, Blind/Fishbowl as signal.
- Cached at `cache/salary-<geo>-<role>.md`. Re-uses if mtime < 90 days unless `--refresh`.
- Confidence-tagged per data point: `Confirmed | Reported | Inferred`.

## Notes

- Cache is per (geo, role). Same role in different markets ⇒ separate cache entries.
- `/career:compare-offer` reads these cache files when comparing a specific offer to market.

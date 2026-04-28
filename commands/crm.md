---
description: External CRM adapters. Sync markdown CRM with Airtable or Attio (two-way), or migrate one-shot between backends. Markdown stays the source of truth.
---

# /career:crm

Stage 9 wiring: CRM adapters. Optional. Use when markdown CRM bites — typically when you want a sortable view, mobile access, or a teammate.

## Subcommands

```
/career:crm sync    --direction=<push|pull|two-way> [--table=<...>] [--dry-run]
/career:crm migrate --from=<markdown|airtable|attio> --to=<...> [--object=<...>] [--dry-run]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `sync`    | `crm-adapter-airtable` or `crm-adapter-attio` (per `CRM_ADAPTER`) |
| `migrate` | `crm-migrate` |

## Notes

- `sync` requires `CRM_ADAPTER` set in config (`airtable` | `attio`).
- `migrate` is a one-shot operation; for ongoing sync, use `sync` after.
- Conflicts are surfaced in `crm/.conflicts-<date>.md` — never auto-merged.
- All external writes confirm before applying.

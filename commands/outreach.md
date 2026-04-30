---
description: Outreach loop entry point — find contact, draft, send, log, and track status. Subcommand-driven. Uses Hunter for contact discovery and email-skills for sending. Always confirms before sending.
---

# /career:outreach

Five sub-skills behind one command. The end-to-end loop: research → contact → draft → confirm → send → log → track.

## Subcommands

```
/career:outreach find         <company> [--role=...] [--n=5] [--append-to-crm] [--lens=...]
/career:outreach find-email   <"Full Name"> <company-or-domain> [--verify] [--append-to-crm] [--lens=...]
/career:outreach verify       <email> [--update-crm]
/career:outreach enrich       <email> [--ad-hoc] [--no-write]
/career:outreach bulk-verify  [--max=50] [--stale-days=90] [--filter=...] [--dry-run]
/career:outreach draft        <company> [--contact-rank=N | --contact="Name email"] [--template=...] [--brief-as=personal|business]
/career:outreach send         <draft-path> [--force]
/career:outreach log          [--company=...] [--contact=...] [--channel=...] [--template=...] [--status=sent] [--date=...] [--note=...]
/career:outreach status       [--latest=<slug> | --row=N | --company=<slug> --contact=<name>] --status=<...> [--note=...] [--next-action=...]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `find` | `find-contact` |
| `find-email` | `find-email-by-name` |
| `verify` | `verify-email` |
| `enrich` | `enrich-contact` |
| `bulk-verify` | `bulk-verify-crm` |
| `draft` | `draft-outreach` |
| `send` | `send-outreach` |
| `log` | `log-outreach` |
| `status` | `track-status` |

## End-to-end example

```
/career:research-company snowglobe                                  # 1. Six briefs + SUMMARY
/career:outreach find snowglobe.ai --role="head of platform"        # 2. Hunter contacts
/career:outreach draft snowglobe --contact-rank=1 --template=cold-pitch  # 3. Draft saved
/career:outreach send drafts/2026-04-28-snowglobe-jane-doe.md       # 4. Confirm + send + log
# ...two days later...
/career:outreach status --latest=snowglobe --status=replied --next-action="reply with follow-up by Wed"
```

## Prerequisites

- `find` requires the `hunter` MCP.
- `send` requires the `email-skills` companion plugin.
- `draft` requires populated `ground-truth.md` (warns and continues if research briefs are missing — drafts will be thinner).
- All write to `${WORKING_FOLDER}` per `${CAREER_DATA_DIR}/config.json`.

## Confirmation discipline

- `find` does not write anything user-facing without explicit `--append-to-crm`.
- `draft` saves to `drafts/` only; never sends.
- `send` requires explicit "yes" + passes through anti-loop check (recent-touch within 14 days blocks unless `--force`).
- `log` appends; never modifies existing rows.
- `status` updates one row at a time and shows the diff before write.

## Notes

- Templates live in `${WORKING_FOLDER}/templates/<template>/`. Each template ships with `EXAMPLE.md` (reference) and may have user-added customised versions beside it (`my-cold-pitch.md` etc) — `draft-outreach` picks `EXAMPLE.md` by default and a custom one if user passes `--template-file=`.
- `find` and `send` are the only subcommands that touch external services. The other three are local file operations.

---
description: Cost-routing + semantic-recall management. Sync the workspace into Pinecone, query it, or inspect the routing decision for a given task.
---

# /career:context

Stage 8 wiring: model routing + semantic recall. Most users won't run this directly — `pinecone-sync` is the only common manual call (after a major workspace edit). The other subcommands are diagnostic.

## Subcommands

```
/career:context sync   [--full] [--dry-run]
/career:context recall --query="..." [--type=<...>] [--top=<n>]
/career:context route  --task=<task-name> [--override=<model>]
```

If invoked without a subcommand, print this help and stop.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `sync`   | `pinecone-sync` |
| `recall` | `semantic-recall` |
| `route`  | `research-router` |

## Notes

- `sync` is idempotent. Run after a heavy ground-truth edit, a discovery run, or weekly.
- `recall` is mostly used internally by other skills; the command exposes it for debugging ("is recall finding the row I think it should?").
- `route` is purely diagnostic — it returns a decision, doesn't actually execute the task.
- Requires `private-misc` companion plugin for Pinecone MCP wiring.

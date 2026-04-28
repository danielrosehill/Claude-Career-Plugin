---
description: Capture entry point — voice memo or text. Transcribes if audio, classifies, proposes route, then on confirmation executes the route (task creation, CRM update, follow-up draft, ground-truth delta proposal). Always confirms before any side-effect.
---

# /career:capture

Two-stage pipeline behind one command: ingest → route. Stage 1 is automatic; stage 2 confirms before any external write.

## Subcommands

```
/career:capture <audio-path>                  # voice ingest + classify + propose route
/career:capture text "<inline text>"          # text input + classify + propose route
/career:capture route <inbox-path> [--reclassify=<class>] [--dry-run]   # execute the route
```

If invoked with just an audio path (no `route` subcommand), runs the ingest stage and prints the proposed route. The user re-runs with `route` to execute.

## Subcommand → skill map

| subcommand | skill |
| --- | --- |
| `<audio-path>` (default) | `voice-ingest` |
| `text "<...>"` | `voice-ingest` (text path bypasses transcription, jumps to classify) |
| `route` | `route-capture` |

## Classifications

The classifier emits one of:
- `quick-note` — single thought / task / fact.
- `retrospective` — recap of an event that happened.
- `strategic-dump` — multi-topic thinking-out-loud.
- `unclear` — too short or genuinely ambiguous.

Each maps to a different route (see `route-capture` skill for details).

## End-to-end example

```
# 1. capture
/career:capture ~/voice-memos/2026-04-28-1430.opus
# → inbox/2026-04-28-1430.md (classification: retrospective)
# → proposes: track-status update + draft follow-up

# 2. route (after user reviews the inbox file)
/career:capture route inbox/2026-04-28-1430.md
# → confirms each side-effect, executes confirmed items, marks file routed
```

## Prerequisites

- Audio path: `claude-transcription` companion plugin installed (run `/career:onboard`).
- Schedule routes: `schedule-manager` companion (falls back to markdown queue if missing).
- Drafts: requires `ground-truth.md` (warns if thin).

## Confirmation discipline

- `voice-ingest` writes only to `inbox/`. No external side-effects.
- `route-capture` confirms each side-effect (task / event / draft / CRM row) individually.
- `--dry-run` on `route` shows the full proposal with no writes.

## Notes

- Inbox files are never deleted. The frontmatter `status` (`pending-route` | `routed` | `parked`) is the source of truth.
- `--reclassify=<class>` overrides the classifier when re-routing.
- Strategic-dump classifications never silently rewrite ground-truth — they propose deltas for the user to apply via `/career:ground-truth edit-section`.

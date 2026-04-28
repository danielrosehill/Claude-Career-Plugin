---
name: voice-ingest
description: Accept an audio file path, transcribe via the claude-transcription companion, classify the transcript (quick-note / retrospective / strategic-dump / unclear), and propose a route. Never auto-routes — always confirms with user. Lands transcript + audio link in <WORKING_FOLDER>/inbox/<YYYY-MM-DD-HHMM>.md.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *), Bash(file *)
---

# Voice Ingest

Take an audio file → transcript → classification → proposed route. Output lands in `inbox/` until the user confirms a route via `route-capture`.

## Inputs

`$ARGUMENTS`:
- **Required**: `<audio-path>` — path to an audio file (opus, mp3, m4a, wav, ogg).
- Optional: `--preset=<preset>` — override transcription preset (default: read from companion).
- Optional: `--engine=<assemblyai|gemini>` — override engine (default: companion config; `gemini-cleaned` recommended for noisy memos).

### Examples

```
/career:capture /home/daniel/voice-memos/2026-04-28-1430.opus
/career:capture ~/Downloads/note.m4a --engine=gemini
```

## Procedure

### 1. Resolve config + paths

`~/.config/career-os/config.json` → `WORKING_FOLDER`, `companions.transcription` (e.g. `claude-transcription`).

If `companions.transcription` is missing, bail with: "no transcription companion installed — run `/career:onboard` and accept the transcription companion offer, or install `claude-transcription` manually."

Validate `<audio-path>` exists. Bail otherwise.

### 2. Transcribe

Delegate to the companion plugin's transcription skill. Default mapping:
- `engine=assemblyai` → `claude-transcription:transcribe-assemblyai`
- `engine=gemini` (default) → `claude-transcription:transcribe-gemini-cleaned`

Pass through `--preset` if provided.

Capture: transcript text, duration (if available), engine used.

### 3. Land transcript in inbox

Path: `${WORKING_FOLDER}/inbox/$(date +%Y-%m-%d-%H%M).md`

Format:

```markdown
---
captured: {{ISO timestamp}}
audio: {{absolute audio path}}
engine: {{assemblyai|gemini-cleaned}}
duration: {{seconds or unknown}}
classification: {{see step 4}}
status: pending-route
---

# Voice memo — {{date}}

## Transcript

{{cleaned transcript}}

## Notes

{{any artifacts the engine returned — speaker labels, timestamps, low-confidence sections}}
```

### 4. Classify

Read the transcript and classify into one of:

- **`quick-note`** — a single thought, task, or fact. Action-shaped or note-shaped. Examples: "remember to follow up with Jane", "Acme just raised series B".
- **`retrospective`** — recap of a meeting, call, or interaction that already happened. Often references a person + an outcome.
- **`strategic-dump`** — multi-topic thinking-out-loud about direction, positioning, ground-truth-shaped content.
- **`unclear`** — too short, too garbled, or genuinely ambiguous.

Update the frontmatter `classification` field.

### 5. Propose a route

Map classification → proposed action (do not execute):

| classification | proposed route |
| --- | --- |
| quick-note | `route-capture` → `schedule-manager:create-task` and/or `crm/companies.md` append |
| retrospective | `route-capture` → `crm/outreach.md` status update + draft follow-up + optional calendar event |
| strategic-dump | `route-capture` → research request enqueued + ground-truth update proposal |
| unclear | leave in inbox; ask user next session |

### 6. Print summary + ask

```
inbox: inbox/<YYYY-MM-DD-HHMM>.md
classification: <class>
proposed route: <route description>

confirm route? (y/n/edit)
  y    → run /career:capture route inbox/<file> --confirm
  n    → leave in inbox
  edit → re-classify or edit transcript first
```

Stop. Do not auto-route. The user runs `route-capture` (or `/career:capture route ...`) explicitly.

## Guardrails

- **Never auto-route in v1.** Always confirm.
- Don't paraphrase the transcript when landing it — keep the engine's output verbatim. Cleanup happens at route time, not ingest time.
- `unclear` is a valid classification, not a failure. The inbox is allowed to hold ambiguous notes.

## Failure modes

- **Audio file missing/unreadable** → bail with the path tested.
- **Transcription companion missing** → bail with install instruction.
- **Transcript empty** → still land an inbox file with `classification: unclear` and a note about engine output.

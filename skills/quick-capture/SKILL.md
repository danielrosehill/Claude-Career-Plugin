---
name: quick-capture
description: Lightweight capture for inline text or short voice notes — same classification + routing pipeline as voice-ingest, but optimized for "one thought, drop it in" rather than full memos. Voice routes through the transcription companion; text skips straight to classify + propose route.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Quick Capture

Same classification + routing as `voice-ingest`, but the input is one thought (text or short voice). Use when you want minimum friction — single command, single thought, single proposed route.

## Inputs

`$ARGUMENTS`:
- **Required (one of)**:
  - `<text-in-quotes>` — one-liner or short paragraph.
  - `<audio-path>` — short voice note.
- Optional: `--route` — auto-confirm the proposed route (still confirms each side-effect within `route-capture`).
- Optional: `--tag=<tag>` — manual tag to bias classification.

### Examples

```
/career:capture quick "remember to follow up with Jane at Snowglobe by Friday"
/career:capture quick ~/voice-memos/short-note.opus
/career:capture quick "salary band at Globex looks below floor" --tag=ground-truth
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`. Inbox dir: `${WORKING_FOLDER}/inbox/`.

### 2. Branch on input type

#### Text input

Skip transcription. Land in inbox immediately:

```markdown
---
captured: {{ISO}}
input-type: text
classification: {{see step 3}}
status: pending-route
tag: {{if --tag}}
---

# Quick capture — {{date}}

{{verbatim text}}
```

#### Audio input

Delegate to `claude-transcription:transcribe-gemini-cleaned` (or assemblyai per config). Same inbox format as `voice-ingest`, but mark `input-type: voice-quick`.

### 3. Classify

Same four-class system as `voice-ingest`:
- `quick-note` (most common for quick-capture)
- `retrospective`
- `strategic-dump` (rare for short input — usually requires more text)
- `unclear`

If `--tag` is provided, bias classification (e.g. `--tag=ground-truth` → bias toward `strategic-dump`).

### 4. Propose route

Same routing matrix as `voice-ingest`. Update inbox frontmatter `classification`.

### 5. If `--route`, immediately invoke `route-capture`

Pass the inbox file path to `route-capture` with normal confirmation flow (each side-effect still confirms — `--route` only auto-confirms the *first* prompt about whether to route at all).

### 6. Print summary

```
inbox: inbox/<file>
classification: <class>
proposed route: <route description>

if --route: routing now (each side-effect will confirm)
otherwise: confirm route? (y/n)
```

## Guardrails

- Same as `voice-ingest`: never auto-route external side-effects.
- Inline text is verbatim — don't paraphrase the user's note.
- `quick-capture` is for *one thought*. If the text is long-form, the user probably wants `voice-ingest` or a strategic-dump path.

## Failure modes

- **Text shorter than 3 words** → still classify, but lean toward `unclear`.
- **Audio too short to transcribe reliably** → land the inbox entry with engine output verbatim and `classification: unclear`.

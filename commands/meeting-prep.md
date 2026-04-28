---
description: One-page pre-call brief — talking points in user's voice, questions to ask, fallback pitch, success criterion. Reads ground-truth + research briefs. Writes to <WORKING_FOLDER>/meetings/.
---

# /career:meeting-prep

Single-skill command. Produces a one-page brief grounded in ground-truth and any existing research.

## Usage

```
/career:meeting-prep <company-or-person> [options]
```

## Options

- `--what-they-want=<text>` — what the counterparty has signalled they want from the call.
- `--date=<YYYY-MM-DD HH:MM>` — when the call is. Defaults to today.
- `--auto-research` — run `/career:research-company` first if no SUMMARY.md exists for the company.
- `--lens=<employer|client|partner>` — frames tone (default: employer).

## Examples

```
/career:meeting-prep snowglobe --what-they-want="hear about my evals work"
/career:meeting-prep snowglobe --auto-research --date="2026-04-30 10:00"
/career:meeting-prep person:"Jane Doe jane@acme.com" --lens=client
```

## Output

`${WORKING_FOLDER}/meetings/<YYYY-MM-DD>-<slug>-prep.md`. Sections:
- Counterparty's likely agenda
- 5 talking points (in user's voice, each cited)
- 3 questions to ask
- 1 "if cornered" fallback pitch
- 1 "what success looks like" outcome
- Ground-truth one-liner reference

## Prerequisites

- Populated `ground-truth.md` (especially `branding` for tone).
- Recommended: `research/companies/<slug>/SUMMARY.md` + the six briefs. Without them, the brief is marked `## Status: thin`.

## Notes

- Every talking point cites a source. No source = `[unsourced — verify before saying]` flag.
- Brief is single-page on purpose. Five talking points / three questions / one pitch / one outcome — the user can hold it in their head walking into the call.

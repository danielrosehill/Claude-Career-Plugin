---
name: meeting-prep
description: Produce a one-page pre-call brief — 5 talking points in user's voice, 3 questions to ask, 1 "if cornered" framing, 1 "what success looks like" line. Reads ground-truth + any existing research briefs. Saves to <WORKING_FOLDER>/meetings/<YYYY-MM-DD>-<company>-prep.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Meeting Prep

Pre-call framing in a single page. Designed for the moment 30 minutes before the call when the user wants to walk in with their head clear.

## Inputs

`$ARGUMENTS`:
- **Required**: `<company-or-person>` — company slug, or `person:Name <email>` if it's a 1:1 not tied to a company.
- Optional: `--what-they-want=<text>` — what the counterparty has signalled they want from the call.
- Optional: `--date=<YYYY-MM-DD HH:MM>` — when the call is.
- Optional: `--auto-research` — if no briefs exist for the company, run `/career:research-company` first.
- Optional: `--lens=<employer|client|partner>` — frames the brief's tone.

### Examples

```
/career:meeting-prep snowglobe --what-they-want="hear about my evals work" --date="2026-04-30 10:00"
/career:meeting-prep snowglobe --auto-research
/career:meeting-prep person:"Jane Doe jane@acme.com" --lens=client
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`. Output: `${WORKING_FOLDER}/meetings/$(date -d "${date}" +%Y-%m-%d)-${slug}-prep.md` (or today's date if `--date` omitted).

### 2. Load context

- `ground-truth.md` — full read.
- If company target: `${WORKING_FOLDER}/research/companies/${slug}/SUMMARY.md` + the six briefs if present.
- If person target: search for prior outreach.md rows + any meetings prep notes for the same person.
- If `--auto-research` and no SUMMARY.md exists: invoke `research-company` first; wait for completion before continuing.

If briefs are missing and `--auto-research` not set: continue but mark the brief as `## Status: thin — no research backing`.

### 3. Synthesize

The brief has five sections, all grounded in loaded context:

**Talking points (5)** — each:
- One sentence, in the user's voice (use ground-truth `branding` cues).
- Tied to a specific fact about the counterparty (cite the source brief or transcript).
- Forward-leaning, not retrospective.

**Questions to ask (3)** — each:
- Open-ended, not yes/no.
- Demonstrates the user has done the work (specific to *this* company, not generic).
- Asks something the user actually wants to know — not flattery questions.

**If-cornered framing (1)** — a 2–3 sentence pitch the user can fall back on if asked "so what do you do?" or "why are we talking?". Built from ground-truth `branding` + the lens.

**What success looks like (1)** — one sentence. Concrete outcome from this call (not "have a good conversation"). Examples: "We agree on a 30-min follow-up to scope an evals audit." "Jane sends an intro to their head of platform."

**Counterparty's likely agenda** (1–2 lines) — what they probably want, based on `--what-they-want` + any research signals. So the user goes in eyes-open about the other side.

### 4. Write the brief

Format:

```markdown
---
target: {{company-or-person}}
lens: {{lens}}
date: {{call date}}
generated: {{ISO timestamp}}
backed-by: {{list of source files: SUMMARY.md, briefs, ground-truth.md}}
---

# Meeting prep — {{target}} ({{date}})

## Counterparty's likely agenda

- ...

## Talking points

1. **{{header}}** — {{one sentence}} _(source: {{file}})_
2. ...
3. ...
4. ...
5. ...

## Questions to ask

1. ...
2. ...
3. ...

## If cornered

> {{2–3 sentence pitch}}

## What success looks like

> {{one sentence}}

## Reference: ground-truth one-liners

- looking-for: ...
- hard-constraints: ...
- (small reminder, not the full ground-truth)
```

### 5. Print summary

```
brief: meetings/<YYYY-MM-DD>-<slug>-prep.md
backing: <n> briefs + ground-truth
status: full | thin (if no research)

next:
  - thin → /career:research-company <slug> then re-run
  - post-call → /career:capture <audio-path> for retrospective
```

## Guardrails

- Talking points must be *the user's voice*, not generic. If `branding` in ground-truth is empty, ask the user before fabricating tone.
- Every talking point cites a source. No source = drop it or mark `[unsourced — verify before saying]`.
- "What success looks like" is a single concrete outcome. If you can't pick one, the meeting purpose is too fuzzy — surface that to the user.
- Don't pad. Five talking points, three questions — fewer is better than weaker ones.

## Failure modes

- **No ground-truth and no research** → bail; brief would be hallucinated.
- **Person target with no prior outreach record** → continue but mark brief as `## Status: cold — limited backing`.

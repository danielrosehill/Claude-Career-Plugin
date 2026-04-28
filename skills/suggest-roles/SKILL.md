---
name: suggest-roles
description: Given ground-truth and (optionally) a target company or domain, suggest role titles/shapes to target. Distinguishes "roles that exist on the market" from "roles I should pitch creating". Writes to <WORKING_FOLDER>/recommendations/roles-<date>.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Suggest Roles

Two flavors:
1. **Existing roles** — what titles match the user's profile in current postings.
2. **Pitch roles** — what shape of role doesn't exist yet but the user could propose creating.

## Inputs

`$ARGUMENTS`:
- Optional: `--company=<slug>` — scope to one company.
- Optional: `--domain=<slug>` — scope to a domain note.
- Optional: `--mode=<existing|pitch|both>` (default `both`).
- Optional: `--n=<count>` (default 10).

### Examples

```
/career:suggest roles
/career:suggest roles --domain=ai-evals --mode=pitch
/career:suggest roles --company=snowglobe --mode=existing
```

## Procedure

### 1. Load context

- `ground-truth.md` (skills, looking-for, branding, hard-constraints).
- If `--company`: relevant research briefs.
- If `--domain`: the domain note.

### 2. Existing-roles pass

WebSearch for live postings matching ground-truth signals (skills × seniority × lens). For each candidate title:
- Frequency in current postings.
- Typical responsibilities.
- Typical comp band (if known).
- Match strength to ground-truth.

### 3. Pitch-roles pass

Synthesize role shapes that:
- Combine multiple ground-truth strengths in a non-standard way.
- Map to a real need visible in the company / domain (cite the signal).
- Are *small enough to propose without a posting* (often individual-contributor or single-hire functions).

Each pitch role must include:
- One-line shape ("Solo AI prompt-engineer embedded with the policy-sim team").
- Why this user (skills tied to the role).
- Why this company / domain (cited need).
- Anchor contact (who would own this hire).

### 4. Write the recommendations file

```markdown
# Role suggestions — {{date}}

> Mode: {{existing|pitch|both}}
> Scope: {{company|domain|none}}
> Generated: {{datetime}}

## Existing roles to target

| title | typical employer shape | match strength | comp signal | source |

## Roles to pitch

| shape | for whom | why-you | why-them | anchor contact |

## Skills gaps the user would need to close for these

- ... (links to /career:suggest skills)

## Sources

1. ...
```

### 5. Print summary

```
recommendations: recommendations/roles-<date>.md
existing: <n>
pitch: <n>

next:
  existing  → /career:outreach find <company> --role="<title>"
  pitch     → /career:suggest consulting-packages --domain=<slug>
```

## Guardrails

- Don't conflate the two modes. An existing role is a posting; a pitch role is a hypothesis.
- Pitch roles must cite a real signal. "They could use someone like this" is not a signal.
- Match strength = honest match to *current* ground-truth. Don't grade on potential.

## Failure modes

- **No live postings + no domain note** → return only `pitch` mode with a warning.
- **ground-truth too thin** → recommend `/career:ground-truth edit` before re-running.

---
name: tailor-resume
description: Produce a resume variant tailored to a specific role/JD. Reads master resume + ground-truth + JD; rewrites bullets to surface relevant experience and mirror JD phrasing where truthful. Never invents experience. Writes to resume/variants/<company>-<role>.md with a tailoring summary.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Tailor Resume

A targeted resume version for one application. The master resume is the truth; this is a *view* of it that emphasises what the JD wants.

**No fabrication.** If the user lacks a JD must-have, leave it out — don't invent.

## Inputs

`$ARGUMENTS`:
- **Required (one of)**:
  - `--jd=<file-path>` — JD as markdown/text.
  - `--jd-url=<url>` — fetched and parsed.
  - `--opportunity=<slug>` — pulls JD from `crm/opportunities.md` row.
- **Required**: `--company=<slug>` and `--role=<slug>` for the output filename.
- Optional: `--master=<path>` — override default master resume location.
- Optional: `--style=<concise|standard|detailed>` — default `standard`.

### Examples

```
/career:tailor-resume --opportunity=globex-ml-eng --company=globex --role=ml-eng
/career:tailor-resume --jd=~/Downloads/jd.md --company=acme --role=staff-pm
```

## Procedure

### 1. Load inputs

- Master resume — search order: `${WORKING_FOLDER}/resume/master.md`, `${WORKING_FOLDER}/user-context/resume.md`, `${WORKING_FOLDER}/resume.md`. Bail if none found.
- `ground-truth.md` — for the user's stated direction, hard constraints, and recent emphasis.
- The JD.

### 2. Analyse the JD

Extract:
- must-have skills.
- nice-to-haves.
- keywords (technologies, methodologies).
- tone (formal / casual / hands-on).
- seniority signals (years, scope, team size).

Surface any must-haves the user *lacks* per ground-truth — those are gaps, not bullets to fabricate.

### 3. Rewrite

Pass over the master resume:
- **Reorder** sections so the most JD-relevant role is up top.
- **Surface** bullets that match must-haves; demote unrelated bullets.
- **Mirror keywords** in bullet phrasing — but only when truthful (don't claim "Rust" if the underlying work was Go).
- **Compress** sections that are tangential to the JD.
- **Cut** sections older than 10 years unless they cover a JD must-have.

`--style=concise` → 1-page target; aggressive cuts.
`--style=standard` → 2-page target; default.
`--style=detailed` → 3+ pages; only for academic/research roles.

### 4. Write

`${WORKING_FOLDER}/resume/variants/<company>-<role>.md`. Top of file:

```markdown
<!--
Tailored: {{date}}
Source: master.md (mtime {{...}})
JD: {{path/url/opportunity}}

## Tailoring choices
- emphasised: {{which roles / bullets surfaced}}
- mirrored: {{which JD keywords picked up}}
- cut: {{sections compressed/removed}}
- gaps surfaced (not on resume): {{must-haves the user lacks}}
-->

# {{user-name}}

…
```

### 5. Print summary

```
variant: resume/variants/<company>-<role>.md
length: <pages estimate>
gaps surfaced: <N>
recommend: <e.g. "address gap on `mlops production` before applying", or "ready to ship">
```

## Guardrails

- **Never invent experience.** Add to the gaps list instead.
- Don't change *facts* (titles, dates, employers). Bullet phrasing is rewritable; the underlying truth is not.
- The JD's tone is a hint, not a mandate — preserve the user's voice.
- Tailoring choices block at the top of the file is mandatory so the user can review.

## Failure modes

- Master resume not found → bail with where to put it.
- JD too short or generic → produce a variant but warn that tailoring is shallow.
- ground-truth absent → still produces a variant, but won't surface gaps; recommend onboarding.

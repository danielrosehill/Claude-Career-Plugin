---
name: draft-outreach
description: Generate an outreach email draft grounded in ground-truth.md, the company research brief, the target contact, and a chosen template (cold-pitch / warm-followup / consulting-pitch / job-followup / interview-followup). Always saves to drafts/ and presents to the user for review. Never sends. send-outreach is a separate skill that requires explicit confirmation.
disable-model-invocation: false
allowed-tools: Read, Write, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Draft Outreach

Builds an email draft from four inputs:
1. Ground-truth (who the user is, what they do, freelance brand if applicable).
2. Company research (the brief outputs in `research/companies/<slug>/`).
3. Target contact (name, role, email — usually from `find-contact`).
4. Template (the markdown template the user wants to base it on).

Always saves to `drafts/` and shows the user for review. Sending is a separate skill (`send-outreach`).

## Inputs

`$ARGUMENTS`:
- **Required**: `<company>` — first positional.
- One of:
  - `--contact-rank=<n>` — pick from the most recent `find-contact` result.
  - `--contact="<name> <email>"` — supply directly.
- Optional: `--template=<cold-pitch|warm-followup|consulting-pitch|job-followup|interview-followup>` (default: based on heuristic — `cold-pitch` for first-touch, `warm-followup` if prior outreach exists in `crm/outreach.md`, `interview-followup` if invoked within 24h of a logged interview event).
- Optional: `--lens=<employer|client|partner>` (default from CRM row, fallback `employer`).
- Optional: `--brief-as=<personal|business>` — overrides ground-truth `freelance-brand` choice. Affects which sender will be used downstream.

### Examples

```
/career:outreach draft acme --contact-rank=1 --template=cold-pitch
/career:outreach draft snowglobe --contact="Jane Doe jane@snowglobe.ai" --template=consulting-pitch
/career:outreach draft myrofish --contact-rank=2  # template auto-picked
```

## Procedure

### 1. Resolve config + paths

`${CAREER_DATA_DIR}/config.json` → `WORKING_FOLDER`, `PREFERRED_EMAIL_SENDER`, `SECONDARY_EMAIL_SENDER`.

Compute slug. Confirm:
- `${WORKING_FOLDER}/ground-truth.md` exists and is populated.
- `${WORKING_FOLDER}/research/companies/${slug}/` has at least `company-overview.md` (warn if not — drafts without research read generic).

### 2. Resolve template

If `--template` provided, use it. Otherwise heuristic:
- Existing `crm/outreach.md` row for this company with `status` in `replied | meeting-booked | meeting-done` → `warm-followup`.
- Logged interview event within 24h (check `meetings/` if present) → `interview-followup`.
- Lens is `client` → `consulting-pitch`.
- Otherwise → `cold-pitch`.

Read the template's `EXAMPLE.md` (and any user-customised template file beside it — `templates/<template>/<custom>.md`).

### 3. Resolve contact

If `--contact-rank=<n>` and a recent `find-contact` result exists in conversation, pick that row. Otherwise prompt user for the explicit contact.

### 4. Read inputs

- Ground-truth: who-i-am, looking-for, domains-of-interest, freelance-brand (if `--brief-as=business`).
- Research: company-overview (mandatory if exists), recruitment-profile (if exists, for hiring context), cultural-fit (if exists, to anchor "why this company specifically").
- Outreach log: confirm no recent outreach to this same contact (anti-loop).

### 5. Draft

Fill the template's `{{variables}}`. Hard rules:
- `{{specific_observation}}` must come from a real, cited fact in the research briefs. If no cite-worthy fact exists, ask the user for one before proceeding.
- `{{your_relevance}}` comes from ground-truth, not from generic boilerplate.
- `{{ask}}` is single, low-stakes, concrete (e.g. "20 minutes next week").
- For `consulting-pitch`: `{{narrow_offer}}` must be a single scope, not a menu.
- Length follows the template's frontmatter `length` constraint. If the draft exceeds it, tighten.

### 6. Save draft

Path: `${WORKING_FOLDER}/drafts/<YYYY-MM-DD>-<slug>-<contact-slug>.md`.

Frontmatter:

```yaml
---
status: draft
template: <template>
company: <company_name>
slug: <slug>
contact:
  name: <name>
  email: <email>
lens: <lens>
sender: <personal|business>
sender_email: <resolved from config + brief-as>
created: <ISO datetime>
sent_at: null
---
```

Body: subject line + email body in plain markdown.

### 7. Present for review

Show the draft to the user with this footer:

```
Draft saved: drafts/<file>

To send:    /career:outreach send drafts/<file>
To edit:    open the file directly, then re-run /career:outreach send
```

Do **not** auto-invoke `send-outreach`. Stop here.

## Guardrails

- Never invent facts. The "specific observation" must trace to a research-brief citation.
- Never auto-send.
- Length budget honoured.
- Single ask.
- Subject lines: short, specific, never click-baity.

## Failure modes

- **Ground-truth missing or thin** → bail with "run `/career:ground-truth` first; drafts without ground-truth read generic and burn the contact."
- **No research briefs for this company** → warn, offer to run `/career:research-brief overview` first or proceed with thin context (user choice).
- **Template missing** → list available templates.
- **Anti-loop hit** (recent outreach to same contact) → block with explanation; user can override with `--force`.

## Idempotency

- Re-running creates a new dated draft file (does not overwrite).
- Editing an existing draft file in place is supported; `send-outreach` reads from disk, not from this skill's output.

---
description: Onboard the user to a career workspace — capture profile, write plugin config, install companion plugins. Variant-aware (career-os | career-planning | job-search | salary-research).
---

# /career:onboard

First-run setup for a newly-provisioned career workspace. Variant-aware. For `career-os` (the unified default), this writes the global plugin config, populates `ground-truth.md`, and offers companion plugins. For the three lite variants, captures a focused profile in `context/profile.md`.

## Steps

### 1. Detect variant

Read workspace `CLAUDE.md`. Identify variant: `career-os`, `career-planning`, `job-search`, or `salary-research`. Tailor the rest of the flow accordingly.

If no variant declared: ask the user, defaulting to `career-os`.

### 2. Career-OS branch (unified)

If variant is `career-os`, run this branch and stop here.

#### 2a. Write global plugin config

Path: `~/.config/career-os/config.json`. If it already exists, read it and offer to update fields rather than overwriting.

Fields:

```json
{
  "WORKING_FOLDER": "<absolute path to this workspace>",
  "CONTEXT_STORE": "markdown",
  "PREFERRED_EMAIL_SENDER": null,
  "SECONDARY_EMAIL_SENDER": null,
  "USER_LOCATION": null,
  "RESEARCH_MODEL": "claude",
  "PINECONE_INDEX": null,
  "CRM_ADAPTER": "markdown",
  "companions": {
    "transcription": false,
    "schedule": false,
    "email": false,
    "social": false,
    "decision": false,
    "pinecone": false
  }
}
```

Ask the user, with these defaults pre-filled:

- `WORKING_FOLDER` — auto-detected from `pwd`.
- `CONTEXT_STORE` — default `markdown`. Other options: `pinecone`, `ragie`. Tell the user `pinecone` requires the `private-misc` companion (offered later).
- `USER_LOCATION` — required. Used by `discover-remote-friendly` and `salary-research`.
- `PREFERRED_EMAIL_SENDER` — `email-skills:send-personal-email` or `email-skills:send-business-email`. Skip if user is not pursuing outreach.
- `RESEARCH_MODEL` — default `claude`. Options: `deepseek-pro`, `deepseek-flash`. Cost optimisation; can change later.
- `CRM_ADAPTER` — default `markdown`. Options: `airtable`, `attio` (require Stage 9 adapters; not available yet — warn if selected).

Use AskUserQuestion for binary/enum fields. Free-text otherwise. Validate path exists for `WORKING_FOLDER`.

Write the JSON. Confirm with the user before overwriting an existing config.

#### 2b. Populate ground-truth

Invoke the `ground-truth` skill in **bulk-edit** mode. It walks the user through every section of `ground-truth.md`. The user may skip sections with "later" — this is expected; partial ground-truth is fine.

#### 2c. Offer companion plugins

Invoke the `install-companion-plugins` skill. Strongly recommend transcription, schedule-manager, email-skills, social-feedback. Optional: decision-evaluation-framework, private-misc.

#### 2d. Print next steps

```
Career-OS workspace ready.

Foundation set:
- Config:       ~/.config/career-os/config.json
- Ground truth: <WORKING_FOLDER>/ground-truth.md (populated)
- Companions:   <list of installed>

Try next:
- /career:research-company <name>      — six-brief deep-dive on a target.
- /career:suggest companies --domain=<your-niche>  — get suggestions grounded in ground-truth + outreach log.
- /career:meeting-prep <company>       — talking points before a call.
- /career:capture <audio-or-text>      — voice-route a memo into CRM / tasks / drafts.
```

### 3. Lite variants (career-planning / job-search / salary-research)

Backward-compat path. No global config file written; profile lives in `context/profile.md` (or `.json`).

#### 3a. Capture profile

Ask (use AskUserQuestion where binary/enum; free-text otherwise):

- Name / pronouns (optional).
- Current role + employer (or "between roles").
- Years of experience and career stage (early / mid / senior / staff+ / exec / pivoting).
- Discipline / function.
- Location + remote preference.
- Currency.

#### 3b. Capture objectives — variant-specific

- **career-planning:** headline 3-year goal, top 3 skills to grow, time horizon, CPD budget, employer study-leave policy.
- **job-search:** target titles, target companies (allowlist / blocklist), must-haves, nice-to-haves, dealbreakers, target start date.
- **salary-research:** roles being researched, markets, comparison purpose (negotiation / career move / market watch), current total comp (optional — enables gap calculations).

#### 3c. Write profile

Save to `context/profile.md` (or `context/profile.json` for salary-research). Structured, machine-readable. Never store secrets.

#### 3d. Print next steps

- **career-planning** → `/career:skill-gap`, `/career:log-cpd`
- **job-search** → `/career:log-role`, `/career:track-application`, `/career:research-company`
- **salary-research** → `/career:salary-benchmark`, `/career:compare-offer`

## Idempotency

- Re-running on a fully-onboarded workspace re-reads config and offers to update fields.
- Re-running with partial ground-truth resumes where the user left off.
- Companion-plugin offers skip already-installed plugins.

## Failure modes

- **No `CLAUDE.md` in workspace** → bail with instructions to run `/career:new-workspace` first.
- **`mkdir ~/.config/career-os` fails** → fall back to `~/.career-os/config.json` and warn the user.
- **User declines all config fields** → write a minimal stub with `WORKING_FOLDER` only; downstream skills will prompt for missing values on first use.

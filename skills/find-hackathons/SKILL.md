---
name: find-hackathons
description: Enumerate hackathons relevant to a domain. Per spec, hackathons are often a direct line to founders — surface ones that double as outreach surface, not just coding events. Writes to <WORKING_FOLDER>/domain-notes/<domain-slug>/hackathons.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Find Hackathons

Hackathons as outreach surface, not just coding contests. A 48-hour event with the founders of 5 companies you want to talk to is often a higher-leverage week than 5 cold emails.

## Inputs

`$ARGUMENTS`:
- **Required**: `<domain>`.
- Optional: `--geography=<region>` — defaults to `USER_LOCATION` + virtual events.
- Optional: `--horizon=<3mo|6mo|12mo>` (default `6mo`).
- Optional: `--prize-floor=<USD>` — filter out tiny / vanity events.
- Optional: `--virtual-only` / `--in-person-only`.

### Examples

```
/career:ecosystem hackathons "ai agents"
/career:ecosystem hackathons "devtools" --geography="EMEA" --horizon=3mo
/career:ecosystem hackathons "policy tech" --virtual-only
```

## Procedure

### 1. Paths

Output: `${WORKING_FOLDER}/domain-notes/${domain-slug}/hackathons.md`.

### 2. Enumerate

Sources: Devpost, MLH, lablab.ai, ETHGlobal (for crypto/web3-adjacent), AngelHack, vendor-sponsored hackathons (OpenAI / Anthropic / Pinecone / etc), local meetup hackathons.

For each event, capture:
- Name + dates.
- Format: virtual / in-person / hybrid.
- Theme alignment to domain.
- Sponsors / judges / mentors — *this is the high-signal field*. Founders, VCs, eng leaders attending in a mentor capacity = direct line.
- Prize structure.
- Team size + solo allowed?
- Submission requirements (so user can size the build).
- Outreach value (low / med / high) — based on who's attending.

### 3. Write the note

```markdown
# Hackathons — {{domain}}

> Domain: {{domain}}
> Geography: {{geography}}
> Horizon: {{horizon}}
> Surveyed: {{date}}

## High outreach value

| event | dates | format | sponsors / judges (the surface) | theme | prize | source |

## Worth doing for the build (low outreach value)

(same columns)

## Skip

| event | reason |

## Sources

1. ...
```

### 4. Print summary

```
note: domain-notes/<domain-slug>/hackathons.md
high outreach value: <n>
build-only: <n>

next: 
  high outreach → /career:plan week (block hackathon dates)
  follow-up → /career:outreach draft <judge-or-sponsor> --template=warm-followup post-event
```

## Guardrails

- "High outreach value" requires named sponsors/judges that match the user's target list. Don't grade up on hype.
- Submission requirements must be honest about scope — hackathon hand-waves ("build something cool with X") rarely produce shippable artifacts.
- Don't recommend events you can't verify dates for. "TBA 2026" is not a recommendation.

## Failure modes

- **No upcoming events** → write empty file with `## Status: nothing in horizon` and recommend re-running quarterly.

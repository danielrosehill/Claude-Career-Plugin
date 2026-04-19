---
description: Onboard the user to a career workspace — capture profile, objectives, constraints.
---

# /career:onboard

First-run setup for a newly-provisioned career workspace. Captures who the user is, what they're aiming at, and the constraints so later commands don't have to re-ask.

## Steps

### 1. Detect variant

Read workspace `CLAUDE.md`. Identify variant (`career-planning`, `job-search`, `salary-research`). Tailor the onboarding questions accordingly.

### 2. Capture profile

Ask (use AskUserQuestion where binary/enum; free-text otherwise):

- **Name / pronouns** (optional).
- **Current role + employer** (or "between roles").
- **Years of experience** and **career stage** (early / mid / senior / staff+ / exec / pivoting).
- **Discipline / function.**
- **Location + remote preference.**
- **Currency.**

### 3. Capture objectives

Variant-specific:

- **career-planning:** headline 3-year goal, top 3 skills to grow, time horizon, CPD budget, employer study-leave policy.
- **job-search:** target titles, target companies (allowlist / blocklist), must-haves, nice-to-haves, dealbreakers, target start date.
- **salary-research:** roles being researched, markets, comparison purpose (negotiation / career move / market watch), current total comp (optional — enables gap calculations).

### 4. Write profile

Save to `context/profile.md` (or `context/profile.json` for salary-research variant). Structured, machine-readable. Never store secrets (don't ask for current salary if the user didn't offer it).

### 5. Next steps

Print which primitives to run next based on variant:

- career-planning → `/career:skill-gap`, `/career:log-cpd`
- job-search → `/career:log-role`, `/career:track-application`
- salary-research → `/career:salary-benchmark`, `/career:compare-offer`

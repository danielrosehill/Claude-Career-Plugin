---
description: Research a target company — fundamentals, team, recent signals — for application prep or decision making.
---

# /career:research-company

Produce a company research brief for a role the user is considering or interviewing for.

## Steps

### 1. Target

Parse `$ARGUMENTS` for company name or slug. Otherwise ask.

### 2. Research

Using public sources only:

- **Fundamentals:** founded, HQ, size, funding stage / public ticker, revenue bracket.
- **Product / business model:** what they sell, to whom, how they make money.
- **Recent signals:** funding rounds, layoffs, exec moves, product launches, press (last 12 months).
- **Culture signals:** Glassdoor themes, engineering blog, leadership interviews. Treat each as signal, not ground truth.
- **Team / manager:** if a hiring manager or team is known, look them up (LinkedIn, blog, talks).
- **Competitors and market position.**
- **Risks:** runway, regulatory, competitive, key-person, technical debt signals.

### 3. Save

`outputs/analysis/company-reports/<company-slug>.md`:

```markdown
# <Company> — research brief

**Generated:** YYYY-MM-DD

## Fundamentals
...

## Product / business
...

## Recent signals (last 12 mo)
...

## Culture signals
...

## Team / manager
...

## Competitors & market
...

## Risks
...

## Questions to ask in interviews
...
```

### 4. Present

Summarise the 3 best signals, 3 worst, and 5 candidate interview questions.

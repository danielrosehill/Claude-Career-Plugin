---
description: Surface relevant conferences/events for the user's discipline, goals, and travel constraints.
---

# /career:find-conferences

Build a conference shortlist aligned with the user's goals.

## Steps

1. Read `context/profile.md` and `goals/`. Pull discipline, goals, and any stated travel/budget constraints.
2. Parse `$ARGUMENTS` for horizon (`next-6mo`, `this-year`, specific region). Default: next 12 months.
3. Propose 6-10 candidate events: major industry conferences, regional events, specialist workshops, vendor summits. Include at least 2 free/virtual options.
4. For each, capture: name, dates, location (or virtual), CFP deadline (if open), ticket cost, travel estimate, expected value (which goal/skill it advances), prior-year review signal.
5. Save to `conferences/shortlist-YYYY.md`.
6. Present the top 3 picks with a one-line rationale each.

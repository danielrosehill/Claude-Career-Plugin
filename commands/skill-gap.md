---
description: Refresh skills/gaps.md by diffing current skill inventory against target-role profile.
---

# /career:skill-gap

Rebuild the gap analysis. Run after any change to `skills/inventory.md` or `skills/target-profile.md`.

## Steps

1. Read `skills/inventory.md` and `skills/target-profile.md`. If either is missing, tell the user to create them first (or run `/career:onboard`).
2. For each skill in the target profile, find the matching row in inventory (fuzzy-match; ask if ambiguous).
3. Compute `gap = required − current` (missing-from-inventory = 0).
4. Categorise:
   - **Critical:** `gap >= 2`
   - **Material:** `gap == 1`
   - **Met:** `gap <= 0`
5. Carry forward existing **Plan** notes from the previous `gaps.md` where the skill still appears.
6. For new critical gaps with no plan, leave **Plan** blank and surface in the summary.
7. Write `skills/gaps.md` with a `**Last refreshed:** YYYY-MM-DD` line.
8. Report: `N critical, M material, K met. New since last refresh: ...`.

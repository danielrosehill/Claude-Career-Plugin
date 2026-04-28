---
name: suggest-skills
description: Identify skill gaps between ground-truth and target roles/companies/domain. Distinguishes "must close to be considered" from "would round out the profile". Writes to <WORKING_FOLDER>/recommendations/skills-<date>.md.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *), Bash(date *)
---

# Suggest Skills

Compares ground-truth to demand signals from the target (role / company / domain) and surfaces gaps with priority.

## Inputs

`$ARGUMENTS`:
- Optional: `--target-role=<title>` — comma-separated titles.
- Optional: `--target-company=<slug>`.
- Optional: `--target-domain=<slug>`.
- Optional: `--horizon=<weeks>` (default 12) — time the user has to close gaps.

At least one target required.

### Examples

```
/career:suggest skills --target-role="staff engineer, ai platform"
/career:suggest skills --target-domain=ai-evals --horizon=8
/career:suggest skills --target-company=snowglobe
```

## Procedure

### 1. Load context

- `ground-truth.md` (current skills + freelance-brand if relevant).
- Target source(s): postings for `--target-role`, briefs for `--target-company`, note for `--target-domain`.

### 2. Extract demand signals

For each target source, list the skills mentioned with frequency. Group:
- Hard skills (languages, frameworks, tools).
- Domain skills (e.g., "RLHF", "compliance frameworks").
- Soft / scope skills (e.g., "leading 0→1", "vendor management").

### 3. Compare against ground-truth

Classify each demand-signal skill:
- `have` — clearly in ground-truth.
- `partial` — adjacent or unproven.
- `gap` — absent.

For each `gap` and `partial`, estimate:
- **Closing effort** (low / med / high) within `--horizon`.
- **Priority** (must / should / nice) based on how often it appears across target sources.

### 4. Write the recommendations file

```markdown
# Skill gap analysis — {{date}}

> Targets: {{role|company|domain summary}}
> Horizon: {{weeks}}
> Ground-truth as of: {{ground-truth-mtime}}

## Must close (within horizon)

| skill | priority | effort | what evidence would count | suggested approach |

## Should close (over horizon)

| skill | ... |

## Nice to have

| skill | ... |

## Already strong (don't oversell, don't ignore)

| skill | where to demonstrate it |

## Sources

1. ...
```

### 5. Print summary

```
recommendations: recommendations/skills-<date>.md
must: <n>
should: <n>
nice: <n>

next: /career:suggest projects --close-gap=<skill>
```

## Guardrails

- "Evidence that would count" must be concrete (a project, a write-up, a contribution) — not "learn X".
- Don't recommend closing every gap. The horizon is real; surface the trade-off.
- `partial` is honest. Don't classify as `have` to flatter the user; don't classify as `gap` to manufacture work.

## Failure modes

- **No target source available** → bail; ask user to specify target.
- **ground-truth too thin** → run `/career:ground-truth edit` first.

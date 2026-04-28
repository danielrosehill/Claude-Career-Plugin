---
name: discover-by-geography
description: Enumerate companies in a domain operating in or hiring within a specific geography (city, country, region). Writes to <WORKING_FOLDER>/domain-notes/geo-<geo-slug>-<domain-slug>.md. Reads outreach.md to mark contact status.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Discover by Geography

Domain ∩ geography. For users who want a list scoped to a city, country, or region — either because they're there, want to be there, or are sizing a local market for consulting work.

## Inputs

`$ARGUMENTS`:
- **Required**: `<geography>` — city, country, or region. Quoted if multi-word.
- **Required**: `<domain>` — the niche.
- Optional: `--n=<count>` (default 20).
- Optional: `--include-offices` — include companies with a local office, even if HQ is elsewhere.
- Optional: `--lens=<employer|client>` — defaults `employer`.
- Optional: `--refresh`.

### Examples

```
/career:discover by-geography "Tel Aviv" "ai infrastructure"
/career:discover by-geography "Israel" "policy tech" --include-offices
/career:discover by-geography "Berlin" "open-source devtools" --lens=client
```

## Procedure

### 1. Resolve config + paths

`WORKING_FOLDER`, `USER_LOCATION` from config. Output: `${WORKING_FOLDER}/domain-notes/geo-${geo-slug}-${domain-slug}.md`.

### 2. Enumerate

Sources:
- Local startup directories (e.g., Startup Nation Central for Israel, Sifted for EU, F6S).
- Local accelerator / VC portfolio pages.
- LinkedIn companies filter by region + industry.
- Local conference/meetup sponsor lists.

Two passes:
1. **HQ in geography** — primary list.
2. **Offices in geography** (only if `--include-offices`) — separate section.

### 3. Cross-reference outreach.md

Mark contact-status.

### 4. Write the note

```markdown
# {{domain}} in {{geography}}

> Domain: {{domain}}
> Geography: {{geography}}
> Include offices: {{true|false}}
> Surveyed: {{date}}

## HQ in {{geography}}

| name | website | stage | hq | hiring locally? | contact-status | notes |

## Offices in {{geography}} (HQ elsewhere)

(only if --include-offices)

## Local ecosystem signals

- Notable accelerators / VCs active in this niche+geo: ...
- Recent funding events: ...
- Conferences / meetups: ...

## Sources

1. ...
```

### 5. Print summary

```
note: domain-notes/geo-<geo-slug>-<domain-slug>.md
hq-local: <n>
offices-local: <n> (if applicable)
never-contacted: <n>
contacted-stale: <n>
contacted-recent: <n>

next: /career:suggest companies --from=geo-<geo-slug>-<domain-slug>
```

## Guardrails

- "HQ in geography" ≠ "incorporated in geography". Use operating HQ, not paper-of-incorporation.
- "Hiring locally?" should be backed by a current posting; mark `unknown` otherwise.
- Don't auto-include the user's location — geography is an explicit input.

## Failure modes

- **Geography too broad** ("Europe") → ask the user to narrow to country or city.
- **Domain too broad** → bail with `chunked-discovery` recommendation.

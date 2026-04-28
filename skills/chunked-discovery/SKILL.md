---
name: chunked-discovery
description: For broad domains ("AI", "fintech", "climate"), segment into sub-niches first, then drill down per segment using discover-by-domain. Writes a parent index plus per-segment notes. Use when discover-by-domain bails out.
disable-model-invocation: false
allowed-tools: Read, Write, WebSearch, WebFetch, Bash(mkdir *), Bash(test *)
---

# Chunked Discovery

Broad domains can't be enumerated honestly in one pass — the long tail goes off-topic before you get coverage. This skill segments first, then drills.

## Inputs

`$ARGUMENTS`:
- **Required**: `<broad-domain>` — e.g. `"AI"`, `"fintech"`, `"climate tech"`.
- Optional: `--segments=<n>` (default 6) — number of sub-niches to derive.
- Optional: `--depth=<count>` (default 10) — companies per segment.
- Optional: `--auto-drill` — run `discover-by-domain` on each segment automatically. Otherwise, write the index and stop.
- Optional: `--refresh`.

### Examples

```
/career:discover chunked "AI"
/career:discover chunked "fintech" --segments=8 --auto-drill
/career:discover chunked "climate tech" --depth=15
```

## Procedure

### 1. Resolve config + paths

Output index: `${WORKING_FOLDER}/domain-notes/chunked-${broad-slug}.md`. Per-segment notes: `${WORKING_FOLDER}/domain-notes/${segment-slug}.md` (same shape as `discover-by-domain` output).

### 2. Derive segments

WebSearch for "landscape", "market map", VC theses, and analyst reports on the broad domain. Synthesize `--segments` non-overlapping sub-niches. For each:
- Name (kebab-case slug + display name).
- One-line definition.
- Distinguishing axis vs adjacent segments (so the user can spot overlap).

Present the segment list to the user and ask for confirmation / edits before drilling.

### 3. Drill (if `--auto-drill`)

For each confirmed segment, invoke `discover-by-domain` with:
- `<domain>` = segment display name
- `--slug` = segment slug
- `--n` = `--depth`

Skip segments where a fresh note already exists (< 30 days old) unless `--refresh`.

### 4. Write the index

```markdown
# Chunked discovery — {{broad_domain}}

> Surveyed: {{date}}
> Segments: {{n}}
> Drilled: {{true|false}}

## Segments

| slug | name | one-liner | drilled note |
| --- | --- | --- | --- |
| ... | ... | ... | domain-notes/<segment-slug>.md (if drilled) |

## How segments differ

- {{seg-a}} vs {{seg-b}}: ...

## Suggested drill order

1. {{slug}} — reason
2. ...

## Sources

1. ...
```

### 5. Print summary

```
chunked index: domain-notes/chunked-<broad-slug>.md
segments: <n>
drilled: <n> (if --auto-drill)
skipped (recent): <n>

next:
  if not drilled → /career:discover by-domain "<segment>" for each
  if drilled    → /career:suggest companies --domain=<slug> per segment
```

## Guardrails

- Segments must be non-overlapping. If two segments ≥30% overlap, merge or re-cut.
- Don't fabricate segments. If the broad domain is genuinely under-segmented (e.g. a frontier with only one shape), say so and recommend `discover-by-domain` directly.
- `--auto-drill` is bulk WebSearch + WebFetch — warn the user about cost before running with `--segments=8 --depth=20`.

## Failure modes

- **Domain still too broad even after segmentation** ("technology") → bail; ask the user to narrow.
- **Drill failure on a segment** → mark that segment as `## Status: partial` in its note; continue with others.

# career

Claude Code plugin: career planning workflow — role logging, offer comparison, application tracking, and white-hat salary benchmarking, with planning/job-search/salary variants.

Part of the [danielrosehill Claude Code marketplace](https://github.com/danielrosehill/Claude-Code-Plugins).

## What you get

### Primitives (globally available once the plugin is installed)

**Cluster primitives:**
- `/career:log-role` — register a role of interest (pipeline, benchmark, or planning)
- `/career:compare-offer` — side-by-side offer comparison across comp / growth / fit
- `/career:track-application` — pipeline state updates (stage, interview, offer, rejection)
- `/career:salary-benchmark` — white-hat salary benchmarking

**Shared workflow commands:**
- `/career:onboard` — first-run setup per workspace variant
- `/career:log-cpd` — quick-log a CPD activity
- `/career:skill-gap` — refresh the gap analysis
- `/career:find-courses` / `/career:find-conferences` — surface candidates to close gaps
- `/career:quarterly-review` — dated plan-vs-actual
- `/career:pipeline-status` — application-pipeline rollup
- `/career:research-company` — public-source company brief
- `/career:tailor-resume` — JD-tailored resume variant (no fabrication)
- `/career:cover-letter` — grounded cover-letter draft
- `/career:interview-prep` — round-specific prep brief

**Agents:**
- `career-strategist` — multi-step planning subagent
- `application-data-manager` — authoritative read/write for `data/processes.json`

### Provisioning skill

- `/career:new-workspace <name> [--variant=career-planning|job-search|salary-research] [--local-only] [--private]`

Scaffolds a new workspace, personalises `CLAUDE.md` from `~/.claude/CLAUDE.md`, and (by default) creates a GitHub repo.

## Variants

- **career-planning** — goals, skills, CPD log, courses/certs/conferences, quarterly reviews.
- **job-search** — pipeline DB, company research, resume/cover-letter tailoring, interview prep.
- **salary-research** — role / company / landscape benchmarks, offer comparisons. White-hat only.

## Pattern

Primitives live in the plugin → globally available from any cwd.
Workspace scaffolds are provisioned as **data** → no `.claude/` tree inside provisioned workspaces.
Plugin updates never touch workspace data.

See [PLAN.md in Claude-Workspace-Reshaping-190426](https://github.com/danielrosehill/Claude-Workspace-Reshaping-190426) for the full pattern spec this plugin follows.

## Sources absorbed

This plugin absorbs and dedupes three prior template repos:

- `Claude-Career-Planning-Template` → `template/career-planning/` + CPD/skills/review primitives
- `Claude-Job-Search-Strategist` → `template/job-search/` + application/resume/interview primitives + `application-data-manager` agent
- `Claude-Salary-Research-Agent` → `template/salary-research/` + salary-benchmark primitive

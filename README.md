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

## Bundled MCP servers

The plugin ships a `.mcp.json` declaring optional MCP servers career-os skills can use. None are installed automatically — Claude Code prompts the user to enable them when the plugin first loads.

| Server | Used by | Required env vars | Notes |
| --- | --- | --- | --- |
| `hunter` | `find-contact`, `outreach` flow | `HUNTER_API_KEY`. Optional: `HUNTER_MCP_COMMAND` (default `npx`), `HUNTER_MCP_PACKAGE` (default `hunter-mcp`) | Override `HUNTER_MCP_PACKAGE` if you use a different Hunter MCP implementation. |

Other capabilities (LinkedIn outreach, email send, transcription, calendar/tasks, semantic store) are resolved at onboard time by the `detect-mcps` skill, which can either reuse an MCP you already have, install a companion plugin, or — if you point it at an aggregated MCP server — index that server's tools and dispatch through it.

### Aggregated MCP servers

If your MCP setup is consolidated behind a single aggregator (e.g. MCP Jungle, a personal proxy, or any router that exposes many upstream tools under one alias), you can pass that alias to `/career:onboard`. The tool inventory is captured to `${CAREER_DATA_DIR}/mcp-inventory.json` and the resolution map prefers tools served by that aggregator before suggesting new MCP installs.

## Workspace environment variable

Skills that write artifacts (CRM rows, briefs, drafts) default their output path to `$CAREER_WORKSPACE`, which resolves to (in order): the explicit `CAREER_WORKSPACE` env var, the `WORKING_FOLDER` field in `${CAREER_DATA_DIR}/config.json`, or `$PWD`. Set `CAREER_WORKSPACE` in your shell profile to anchor career-os to a specific workspace from any cwd.

## Sources absorbed

This plugin absorbs and dedupes three prior template repos:

- `Claude-Career-Planning-Template` → `template/career-planning/` + CPD/skills/review primitives
- `Claude-Job-Search-Strategist` → `template/job-search/` + application/resume/interview primitives + `application-data-manager` agent
- `Claude-Salary-Research-Agent` → `template/salary-research/` + salary-benchmark primitive

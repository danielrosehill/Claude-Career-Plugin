---
name: new-workspace
description: Provision a new career workspace on disk. Use when the user wants to start a new career-planning, job-search, or salary-research project. Accepts a workspace name and a variant (career-planning | job-search | salary-research). Scaffolds the workspace, personalises CLAUDE.md from the user's global memory, and (by default) creates a GitHub repo.
disable-model-invocation: true
allowed-tools: Bash(mkdir *), Bash(cp *), Bash(cat *), Bash(git init *), Bash(git add *), Bash(git commit *), Bash(gh repo create *), Bash(gh auth status), Bash(git push *), Read
---

# Provision Career Workspace

Creates a new workspace for career planning, job searching, or salary research. This plugin's commands (`/career:log-role`, `/career:compare-offer`, `/career:track-application`, `/career:salary-benchmark`, etc.) are globally available once installed — this skill only provisions the **data scaffold** those commands read from and write to.

## Arguments

`$ARGUMENTS` is parsed as:

- **First positional**: workspace name (kebab-case). Required.
- **Second positional** (optional): target parent path. Defaults to `~/repos/github/my-repos`.
- **`--variant=<career-planning|job-search|salary-research>`** (optional): which scaffold to copy. Default: `career-planning`.
- **`--local-only`** (optional): skip GitHub repo creation and push. Default: create a public GitHub repo and push.
- **`--private`** (optional): create the GitHub repo as private. Default: public.

### Examples

```
/career:new-workspace my-cpd
/career:new-workspace 2026-job-hunt --variant=job-search
/career:new-workspace staff-eng-comp --variant=salary-research --private
```

## Procedure

### 1. Parse arguments

Extract workspace name, target parent path, variant, and flags from `$ARGUMENTS`. If the workspace name is missing, ask for it before proceeding. If variant isn't one of the three, list the available variants.

### 2. Resolve the scaffold path

The bundled scaffold lives at `${CLAUDE_SKILL_DIR}/../../template/<variant>/`. Confirm it exists.

### 3. Read ambient facts

Read `~/.claude/CLAUDE.md` if it exists. Extract OS, locale, timezone, and user identity facts to personalise the workspace CLAUDE.md.

### 4. Create the workspace directory

```bash
mkdir -p <target-parent>/<workspace-name>
cp -r ${CLAUDE_SKILL_DIR}/../../template/<variant>/. <target-parent>/<workspace-name>/
```

Do **not** copy any `.claude/` tree. The plugin's primitives are global.

### 5. Personalise CLAUDE.md

Open the new workspace's `CLAUDE.md` and:

- Replace placeholder identity (`{{PROJECT_NAME}}`, `{{DESCRIPTION}}`, `{{ROLE}}`, `{{STAGE}}`, `{{HEADLINE_GOAL}}`) with facts from step 3 or prompt the user for any missing ones.
- Add a short header noting the workspace name and variant.
- Embed ambient OS/locale/timezone so downstream commands can skip re-asking.

### 6. Prompt for variant-specific facts

Ask only for facts this plugin can't infer:

- **career-planning:** current role, career stage, headline 3-year goal.
- **job-search:** target titles, target start date, hard constraints (remote only / location / salary floor if user wants it stored).
- **salary-research:** role(s) to benchmark, market/geo, currency.

Write them into the workspace `CLAUDE.md` and/or `context/profile.md` (or `context/profile.json` for salary-research).

### 7. Initialise git and (optionally) publish

```bash
cd <target-parent>/<workspace-name>
git init
git add .
git commit -m "Initial workspace from career plugin"
```

Unless `--local-only` is set:

```bash
gh repo create <workspace-name> --<public|private> --source=. --push
```

Use `--public` by default, `--private` if the flag was passed. **Strongly encourage `--private`** for job-search and salary-research workspaces — they typically contain sensitive material — but never override the user's explicit choice.

### 8. Print next steps

Tell the user:

- Workspace path and variant.
- Which plugin commands apply:
  - **career-planning:** `/career:onboard`, `/career:skill-gap`, `/career:log-cpd`, `/career:find-courses`, `/career:find-conferences`, `/career:quarterly-review`.
  - **job-search:** `/career:onboard`, `/career:log-role`, `/career:track-application`, `/career:pipeline-status`, `/career:research-company`, `/career:tailor-resume`, `/career:cover-letter`, `/career:interview-prep`.
  - **salary-research:** `/career:onboard`, `/career:salary-benchmark`, `/career:compare-offer`.
- Reminder that the workspace is **data** — they can delete/move/archive it without losing the plugin's commands.

## Notes

- Resolve the scaffold via `${CLAUDE_SKILL_DIR}/../../template/` (not `${CLAUDE_PLUGIN_ROOT}` — only exported in hooks/MCP).
- Never copy `.claude/commands/`, `.claude/agents/`, or `.claude/skills/` into the new workspace.
- No hard-coded personal paths or identifiers here — everything comes from user memory or prompts.

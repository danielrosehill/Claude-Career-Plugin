---
name: install-companion-plugins
description: Offer to install Claude Code plugins that complement the career plugin. Use during /career:onboard, or whenever the user wants to enable a downstream capability (transcription for voice ingest, schedule-manager for calendar/tasks, email-skills for outreach send, social-feedback for company sentiment research, decision-evaluation-framework for offer comparisons, private-misc for Pinecone wiring). Lists each, explains what it unlocks in the career plugin, and lets the user accept/decline per plugin.
disable-model-invocation: false
allowed-tools: Bash(claude plugin *), Bash(claude marketplace *), Read
---

# Install Companion Plugins

The career plugin deliberately does not reimplement transcription, scheduling, email sending, social-sentiment research, or decision frameworks. Those live in dedicated plugins. This skill offers them at onboarding time and at any later point when the user wants to enable the corresponding capability.

## Companion plugins

| Plugin | What it unlocks in career-os | Used by |
| --- | --- | --- |
| `claude-transcription` | Voice-memo ingest pipeline (AssemblyAI / Gemini transcribers). | `voice-ingest`, `capture`, retrospectives |
| `schedule-manager` | Google Calendar + Todoist for tasks and meeting blocks. | `weekly-plan`, `meeting-prep`, `route-capture` |
| `email-skills` | Send outreach emails from personal or business address. | `send-outreach` |
| `social-feedback` | Reddit / HN / Trustpilot / YouTube discourse for company sentiment. | `brief-glassdoor-signal`, `brief-cultural-fit`, `brief-recruitment-profile` |
| `decision-evaluation-framework` | Multi-lens evaluation of offers, pivots, big career calls. | `compare-offer`, optional in `meeting-prep` |
| `private-misc` | Pinecone MCP wiring for semantic context recall. | Stage 8 `pinecone-sync`, `semantic-recall` |

## When to invoke

- Final step of `/career:onboard` (always).
- User asks "what plugins should I install?" or "what's missing?"
- A skill detects a missing dependency and asks the user — that skill calls this one with a focused subset (e.g. `--only=email-skills`).

## Procedure

### 1. Detect already-installed

Run `claude plugin list` (or equivalent) and parse the output. For each companion plugin in the table above, mark `installed | not-installed`.

### 2. Present the menu

For each not-installed plugin, show:

- Name.
- One-line "what it unlocks here."
- Recommendation level: **strongly recommended** (transcription, schedule-manager, email-skills, social-feedback) or **optional** (decision-evaluation-framework, private-misc).

Use `AskUserQuestion` per plugin: `Install {{plugin}}? (yes / no / later)`.

### 3. Install accepted plugins

For each `yes`, run the install command. Do not batch — surface failures per-plugin so the user knows which succeeded.

```bash
claude plugin install <plugin-name>
```

If the marketplace alias differs (e.g. namespaced plugins under a vendor prefix), discover the correct command from `claude marketplace search <plugin>`.

### 4. Trigger first-run setup where applicable

After install, several companion plugins need their own onboarding:

- `claude-transcription` → `/claude-transcription:configure`
- `schedule-manager` → `/schedule-manager:onboard`
- `email-skills` → no setup, uses ambient SMTP / Resend env.
- `social-feedback` → no setup.
- `decision-evaluation-framework` → `/decision-evaluation-framework:onboard`
- `private-misc` → `/private-misc:personal-context-injection`

Offer to run each one now or later.

### 5. Update career-os config

For each installed plugin, persist a flag in `${CAREER_DATA_DIR}/config.json` under `companions`:

```json
{
  "companions": {
    "transcription": true,
    "schedule": true,
    "email": true,
    "social": false,
    "decision": false,
    "pinecone": false
  }
}
```

Other career skills check this map before delegating — if a companion is missing, they degrade gracefully (e.g. `voice-ingest` without `transcription` falls back to "paste the transcript yourself").

### 6. Print summary

```
Installed: claude-transcription, schedule-manager, email-skills
Skipped:   social-feedback (later)
Already:   none
Next:      run /claude-transcription:configure when ready
```

## Arguments

`$ARGUMENTS`:

- `--only=<plugin1,plugin2>` — only offer this subset.
- `--quiet` — skip the explanation paragraph; show the menu directly.
- `--no-setup` — install only; don't trigger first-run setups.

## Failure modes

- **`claude plugin` not available** — bail with instructions to upgrade Claude Code.
- **Plugin install fails (network, marketplace miss)** — show the error verbatim, mark plugin as `not-installed` in config, offer retry / skip.
- **User declines all** — that's fine; record skips so onboarding doesn't re-ask next time.

## Idempotency

- Re-running detects already-installed plugins and skips them.
- Re-running after a previous "later" offers them again.

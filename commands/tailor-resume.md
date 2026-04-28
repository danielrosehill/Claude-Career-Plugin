---
description: Generate a resume variant tailored to a specific role/JD. Never invents experience.
---

# /career:tailor-resume

Single-skill command. Runs the `tailor-resume` skill.

## Usage

```
/career:tailor-resume --opportunity=<slug> --company=<slug> --role=<slug>
/career:tailor-resume --jd=<file>      --company=<slug> --role=<slug>
/career:tailor-resume --jd-url=<url>   --company=<slug> --role=<slug>
```

## What it does

- Reads master resume + ground-truth + JD.
- Surfaces JD-relevant experience; mirrors keywords *where truthful*; cuts tangential bullets.
- Surfaces gaps (JD must-haves the user lacks) — never fabricated as bullets.
- Writes to `resume/variants/<company>-<role>.md` with a tailoring summary at the top.

## Notes

- Master resume search order: `resume/master.md`, `user-context/resume.md`, `resume.md`.
- Style flag: `--style=concise|standard|detailed` (default standard).
- Combine with `/career:skill-gap` to address surfaced gaps before applying.

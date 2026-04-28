# Resume

Master resume + per-application variants live here.

## Convention

```
resume/
  master.md              # canonical CV — the version of record
  variants/
    <company>-<role>.md  # tailored variants produced by /career:tailor-resume
```

`master.md` is hand-edited. Variants are generated; safe to delete and regenerate.

`tailor-resume` (Stage 10) reads `master.md` + a target role brief + `ground-truth.md` and writes a new variant. It diffs cleanly against `master.md`.

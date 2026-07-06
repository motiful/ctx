# ctx skeleton (asset)

The canonical `/ctx` folder shape. `ctx-adopt` copies this skeleton, then the agent FILLS each
README's placeholders per project (structure is fixed = the standard; content is per-project).

| folder | lifetime | holds |
|---|---|---|
| `spec/` | LIVING | current truth — what we're building. Edit in place. Start with `spec.md` (Reference-mode). Written via ctx-spec.
| `decisions/` | APPEND-ONLY | ADRs — a choice + why, numbered, never rewritten (supersede in place). Written via ctx-spec. Index in README.
| `progress/` | LIVING | where we are, what's next, what to read first. One `progress.md` until ~150 lines, then a tree. Via ctx-progress.
| `reports/` | DISPOSABLE | numbered HTML a human decides from, then distilled into the SOT and archived. Via ctx-report. NOT a scan entry.
| `scratch/` | DISPOSABLE | raw notes / research / experiments. No format contract. Archived aside, never a scan entry.

Fill order: seed `spec/` (current truth) + `decisions/` (≥ the seed ADRs), then keep `progress/`
live. `reports/` and `scratch/` fill as work happens. Done = the ctx-folder DoD checklist in the
ctx skill passes.

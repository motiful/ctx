---
name: ctx-report
description: Produces a disposable HTML report laid out for a human to read, comment on, and decide from — then distills the keep-worthy conclusions into the source of truth and archives the report. Use when writing a report, a review doc, or an options-for-a-decision document for a human to weigh in on in a ctx knowledge base, when a human has commented on a report and it needs distilling, or when un-merged reports are piling up.
license: MIT
metadata:
  author: motiful
  version: "1.1"
---

# ctx-report — write a report a human decides from, then dispose of it

> A **report is DISPOSABLE** (lifetime model in [`../ctx`](../ctx/SKILL.md)): it lays thinking out for a human to review; its keep-worthy conclusions distill into the SOT (`spec/` + `decisions/`), then it is archived-aside. It is NOT a source of truth.

## What a report is (and is not)

`ctx/reports/` is where thinking is laid out as **HTML for a human to read and comment on** — numbered, indexed, rolling-merged, archived (not deleted). It carries the visible "chaos → converge" output of the loop, but is **NOT a source of truth**; the truth ends up in `spec/` and `decisions/`.

- Raw research and notes do NOT live here — they live in `scratch/`. Research **surfaces** as a report only when shown to a human; once reviewed, its keep-worthy conclusions distill into the SOT and the report is disposable.
- A report is **disposable and NOT a scan entry**: open one only when a SOT doc links to it — never deep-scan `reports/` to understand the project (it holds superseded/rejected thinking that surfaces as false positives).

## Execution Procedure

```
produce_or_distill_report(task) → decision-ready HTML, then disposed of

# STEP 1 — Write (when thinking must be surfaced for a human to review/decide)
if surfacing thinking for a human:
    write_html_report(task)              # plain language · terms defined inline · what+how first, why later · structurally self-verified
    end_with_verdict_list(task)          # "for you to decide" + options, gated by confidence × blast-radius
    use_absolute_paths(anything the reader opens)

# STEP 2 — Distill (after the human comments) — so nobody tracks what was reviewed
for comment in human_comments:
    bucket = classify(comment)           # keep | drop | unsure
    if bucket == keep:   Skill("ctx-merge", conclusion)   # sink to spec/decisions with provenance
    if bucket == drop:   record_reject(reason)            # reject-log home = decisions/ (see ctx-merge)
    if bucket == unsure: carry_to_next_report(comment)    # into the next "to decide" list

# STEP 3 — Rolling-merge (stop the pile becoming a new mess)
if count(unmerged_reports) >= ~5 or cluster_ready:
    offer_merge()                        # ASK first; a summary S<lo>-<hi>.html becomes the new baseline
    keep_index_in_sync("reports/README") # consistency.md Rule 1

# STEP 4 — Archive-aside (on completion)
move_unit_verbatim("reports/archive/")   # non-SOT; numbers unchanged, never reused

# Before committing — apply the cross-cutting constraints
apply("../ctx/references/consistency.md")   # single-source (index sync) · same-change · verify-canonical · gate
```

Each `Skill()` above is a decision-layer entry — enter the module when the step runs.

## How to write a report (default format)

Default output is **HTML** (unless the task asks for video / markdown). Every report:

1. **Plain language.** Write for a smart non-specialist; don't stack jargon.
2. **Anchor each answer to its question.** A conclusion is only verifiable against the question it answers — a reader who lost the question can't judge whether the answer is right. Make the driving question explicit, at a granularity that fits the report's shape: a **response-to-asks** report (a review reply, a decision request) quotes each section's originating question **verbatim** from the asker at the section head; a **single-topic** narrative / audit states the driving question + trigger in the top framing. Not every report is one-question-per-section. (Mirrors rule 6: open anchored to the question you answer, close anchored to the question you still ask.)
3. **Terms defined inline.** When a term appears, define it in a dedicated block (CN/EN + usage + source), not in passing. A report explains, so first-use definitions are required — unlike a spec (normative, terse), which may use a term undefined and link out.
4. **What + how first, why later.** Open with what's proposed and how it works; put rationale in later sections.
5. **Self-contained + structurally self-verified.** DOCTYPE, balanced tags, anchors aligned. After writing, verify byte/tag counts — do not trust "looks right".
6. **End by asking for a verdict.** Close with an explicit "for you to decide" list + options, so the reader gives a clear judgment instead of hunting for the open points. **Gate what lands in that list by confidence × blast-radius:** high-confidence + low-blast findings you can just state as done/recommended; anything high-blast OR low-confidence is what you escalate here as an explicit decision for the human. Don't bury a colossal-or-uncertain call in prose, and don't pad the verdict list with reversible things you were sure of.
7. **Absolute paths for anything the reader opens.** Every path the report shows the human to open — the markdown/spec files a review links to, deliverables, "look at X" pointers — MUST be absolute (`/Users/…` or a `file:///…` URI), never `./`/`../` relative or a bare filename. The reader is in a terminal/browser where relative paths are not clickable, and a report is disposable — it carries no cwd context. (Cross-references *between* ctx docs stay repo-relative; this rule is only for paths surfaced to a human.)

## After a comment: distillation (so nobody has to track what was reviewed)

When the human comments, split it into three:

| Bucket | Handling |
|---|---|
| **keep** (approved) | distill → sink to `spec/` (current truth) or `decisions/` (a choice + why), via **ctx-merge**'s disposition ledger, each with provenance. |
| **drop** (rejected) | mark removed + one-line reason; do not sink. Record in the reject log (its home is `decisions/` — see **ctx-merge**) so it can't quietly return under a new name. |
| **unsure** | carry as an open question into the next report's "to decide" list; keep researching. |

## Rolling-merge (stop the pile becoming a new mess)

- **Iterate on the latest.** Improve the highest-numbered report rather than spawning a fresh one each time.
- **Proactively offer to merge** when un-merged reports reach **~5** (or a cluster is ready to close). Ask before summarizing.
- **Summary = the new baseline.** Merge a range into one `S<lo>-<hi>.html` (e.g. `S47-53.html`); it becomes the base for further questions. Then move the merged sources into `reports/archive/` (numbers unchanged, never reused).
- **Keep a status view** ("what's sunk / open / where the latest is") in `ctx/README.md` (or `reports/README.md`) — it drives progress control.

## Index & archive

- `reports/` is non-SOT, so its "what exists / what status" lives in a README index, kept in sync on every add/remove (**consistency.md** Rule 1 gate).
- A report may be a file or a folder (per conventions); branches get subfolders. On completion, archive-aside: move the whole unit verbatim into `reports/archive/`.

## The three lifetime classes

Not restated here — see **`../ctx/SKILL.md § the model`**. `reports/` is **DISPOSABLE**. Hard constraints (index sync, disposable-not-scan-entry) live in **consistency.md** (Rule 1); this skill teaches the method.

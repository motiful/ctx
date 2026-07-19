---
name: ctx-report
description: Produces a disposable HTML report laid out for a human to read, comment on, and decide from — then distills the keep-worthy conclusions into the source of truth and archives the report. Use when writing a report, a review doc, or an options-for-a-decision document for a human to weigh in on in a ctx knowledge base, when a human has commented on a report and it needs distilling, or when un-merged reports are piling up.
license: MIT
metadata:
  author: motiful
  version: "1.5"
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

# STEP 0 — Structure the ask, verify premises, work it toward a recommendation (before any drafting)
# Scope: the whole interaction — reading the ask, researching, deciding what to escalate — not just the HTML artifact.
if surfacing thinking for a human:
    structure(task)                      # STRUCTURE: 梳理 MECE-exhaustive sub-issues + 发掘 laddering to the root question — see § STEP 0
    verify_premises(task)                # date-tool + current year · resolve-hypocognition-not-mirror · value-elicitation-upfront — see § STEP 0
    work_it(task)                        # exhaust what's researchable → converge to one recommendation → escalate only the human's private call, with a recommendation attached — see § STEP 0

# STEP 1 — Write (when thinking must be surfaced for a human to review/decide)
if surfacing thinking for a human:
    ground_first(task)                   # research-first: read/verify what the claims rest on BEFORE drafting — a report is the decision layer; never write from memory
    write_html_report(task)              # plain language · terms defined inline · question-map opens it · what+how first, why later · structurally self-verified
    end_with_verdict_list(task)          # "for you to decide" + options, gated by confidence × blast-radius
    use_absolute_paths(anything the reader opens)

# STEP 2 — Distill (after the human comments) — so nobody tracks what was reviewed
for comment in human_comments:
    bucket = classify(comment)           # keep | drop | unsure
    if bucket == keep:   Skill("ctx-merge", conclusion)   # sink to spec/decisions with provenance
    if bucket == drop:   record_reject(reason)            # reject-log home = decisions/ (see ctx-merge)
    if bucket == unsure: carry_to_next_report(comment)    # into the next "to decide" list

# STEP 3 — Archive-aside (state-driven: a report is SPENT once reviewed + distilled)
if human_reviewed(report) and keep_worthy_distilled_to_SOT(report):   # its job is done — the facts now live in spec/decisions
    move_unit_verbatim("reports/archive/")   # non-SOT; numbers unchanged, never reused
    collapse_index_row("reports/README")     # index lists LIVE reports only; archived → one pointer line (consistency.md Rule 1)

# STEP 4 — Batch sweep (ONLY a historical backlog piled up un-archived, or a phase closing)
if a cluster of already-reviewed reports accumulated or a phase closed:
    offer_batch_archive()                # ASK first; sweep the spent cluster into archive/ in one move
    if the pile is worth one synthesis: offer_merge("S<lo>-<hi>.html")   # optional summary baseline

# Before committing — apply the cross-cutting constraints
apply("../ctx/references/consistency.md")   # single-source (index sync) · same-change · verify-canonical · gate
```

Each `Skill()` above is a decision-layer entry — enter the module when the step runs.

## STEP 0 — structure the ask, verify premises, work it

### STRUCTURE (before naming anything)

Pair two moves, not a single umbrella technique:

- **梳理 (enumerate)** — MECE: lay out the sub-issues so they're mutually exclusive and collectively exhaustive; no silent gap.
- **发掘 (ladder)** — abstraction-laddering: dig from the surface question down to the root question it's really standing in for.

Before naming any concept STRUCTURE surfaces, run it through a **term-operation-consistency-gate**: does the term's full established meaning conflict with the operation you want it to carry? If yes, drop the term and keep only the operation — a term whose baggage fights the rule it's attached to is worse than an unlabeled rule.

### Resolve hypocognition, don't mirror

When the human names a felt-but-wordless gap in their own improvised words, find the established term for it and use that — don't parrot their phrase back dressed up as an answer. Anti-sycophancy applies the same way to a claim: verify it, don't rubber-stamp it because the human stated it with confidence.

The reverse also holds: when the human's own phrasing is close to but not quite one of this collection's controlled terms (argument-to-moderation, graceful degradation, decision-forcing, and the like — all already inline in this doc), map it to the precise term before answering. The mapping is cheap and needs no separate lookup — do it inline, the same way you'd resolve any other imprecise phrasing.

### Verify premises

Before drafting: confirm the current date, separate what's a checkable fact from what's a subjective call, and — if a private-preference input (budget, audience, goal) would make the downstream research moot without knowing it — ask that ONE question upfront, before starting, not after a wasted pass.

### Decision-forcing (don't stop at "it depends")

When a question has more than one reasonable answer:

1. **MUST exhaust what's researchable first.** Anything you can look up, compute, or reason through is your job, not the human's — don't hand back an open question a few more minutes of work would close.
2. **MUST converge to one ranked recommendation.** State "X, because Y" — not a menu of options with no pick attached. Weighing every option and refusing to choose (**argument to moderation**) is exactly the failure this rule blocks.
3. **MUST escalate only the part that is genuinely the human's call** — private preference, risk tolerance, budget, taste — and MUST attach your own recommendation when you do. Never hand back a blank "you decide."

**Graceful degradation, true and false.** Borrowed from fault tolerance: a system under partial failure keeps serving at reduced scope instead of going fully dark. True graceful degradation is fine — a smaller, honestly-labeled partial result beats nothing. **False graceful degradation is banned**: dressing up a full stop as if it were a controlled reduction. Two shapes of it — quietly under-delivering without saying so (**graceful-skip**), and halting or punting a decision you had the means to research or commit to (**graceful-stop**) — both look composed on the surface while actually withholding delivery or handing your own cost back to the human.

## How to write a report (default format)

> **Ground it before you draft it (research-first).** A report is the **decision layer** — its conclusions distill into the SOT, so an ungrounded claim propagates a wrong decision downstream. Do the reading/verification the claims rest on *first*; never write a report from memory or assumption. (Raw research lives in `scratch/` and *surfaces* as a report only once it's grounded — per *What a report is* above.)

Default output is **HTML**, and this holds **wherever the user says "report" — ctx project or not.** "Report" names a *format*, not a folder: even a one-off in a repo with no `ctx/` store gets the full HTML treatment below, because the reader still deserves a decision-grade, human-readable artifact (a bare `.md` dump is the failure mode this skill exists to prevent). Drop to another format only when the task *explicitly* asks ("give me markdown / a video"). What is ctx-project-scoped is only the `reports/` **folder lifecycle** (index · state-driven archive) — outside a ctx store a report is just a standalone HTML file with no folder apparatus, still written to this standard. Every report:

1. **Plain language.** Write for a smart non-specialist; don't stack jargon.
2. **Anchor each answer to its question.** A conclusion is only verifiable against the question it answers — a reader who lost the question can't judge whether the answer is right. Make the driving question explicit, at a granularity that fits the report's shape: a **response-to-asks** report (a review reply, a decision request) quotes each section's originating question **verbatim** from the asker at the section head; a **single-topic** narrative / audit states the driving question + trigger in the top framing. Not every report is one-question-per-section. (Mirrors rule 6: open anchored to the question you answer, close anchored to the question you still ask.)
3. **Terms defined inline.** When a term appears, define it in a dedicated block (the reader's working language + the term's canonical original form + usage + source — tech terms default to English; culture-specific terms keep their source-culture original), not in passing. A report explains, so first-use definitions are required — unlike a spec (normative, terse), which may use a term undefined and link out. This also covers a pair of words you coin yourself to imply a contrast — if you invent two verbs to stand in for two different treatments, spell out each one's subject and object in the same sentence you introduce them; a coined pair that reads as clear to its author but never states what's being contrasted looks explained without being explained.
4. **What + how first, why later.** Open with what's proposed and how it works; put rationale in later sections.
5. **Self-contained + structurally self-verified.** DOCTYPE, balanced tags, anchors aligned. After writing, verify byte/tag counts — do not trust "looks right".
6. **End by asking for a verdict.** Close with an explicit "for you to decide" list + options, so the reader gives a clear judgment instead of hunting for the open points. **Gate what lands in that list by confidence × blast-radius:** high-confidence + low-blast findings you can just state as done/recommended; anything high-blast OR low-confidence is what you escalate here as an explicit decision for the human. Don't bury a colossal-or-uncertain call in prose, and don't pad the verdict list with reversible things you were sure of.
7. **Absolute paths for anything the reader opens.** Every path the report shows the human to open — the markdown/spec files a review links to, deliverables, "look at X" pointers — MUST be absolute (`/Users/…` or a `file:///…` URI), never `./`/`../` relative or a bare filename. The reader is in a terminal/browser where relative paths are not clickable, and a report is disposable — it carries no cwd context. (Cross-references *between* ctx docs stay repo-relative; this rule is only for paths surfaced to a human.)
8. **Open with a question-map, not a table of contents.** Before any prose, run STRUCTURE's own algorithm on the report's own content and surface it as a diagram — root question at the top, every branch a genuinely intriguing sub-question a reader wants answered, not a section label that only makes sense after reading the section. Symmetric/linear structure → ASCII `<pre>`. Asymmetric/branching structure → mermaid.js via CDN (`<script type="module">` + jsdelivr; renders in a bare `file://`-opened HTML, no build step), laid out **horizontally** (`flowchart LR`) and wrapped with a small CDN pan-zoom library over the rendered SVG — cramped default sizing otherwise makes node text unreadable. Skip the diagram only when STRUCTURE's own laddering collapsed to one question with no real branches — state that single question as a callout instead of forcing an empty tree.

## After a comment: distillation (so nobody has to track what was reviewed)

When the human comments, split it into three:

| Bucket | Handling |
|---|---|
| **keep** (approved) | distill → sink to `spec/` (current truth) or `decisions/` (a choice + why), via **ctx-merge**'s disposition ledger, each with provenance. |
| **drop** (rejected) | mark removed + one-line reason; do not sink. Record in the reject log (its home is `decisions/` — see **ctx-merge**) so it can't quietly return under a new name. |
| **unsure** | carry as an open question into the next report's "to decide" list; keep researching. |

## Archiving — state-driven (a report is spent once reviewed + distilled)

The distillation step already records the keep-worthy facts into the SOT as you review (keep → sink; drop → reject-log). So **a report's job is done the moment it has been reviewed by the human and its keep-worthy conclusions distilled** — from then on it is a superseded rationale trail, not a live surface. Archive it then; don't wait for a count.

- **Archive per-report, on "spent".** Reviewed + distilled → move the unit into `reports/archive/` and collapse its index row to a pointer. The index lists only **live** reports (awaiting review, or an open decision surface) — that keeps the index (which carries the token cost) lean.
- **Archive by thread, not by report, when one report seeded several.** A report is spent as a whole only once *every* question it opened has converged — if it forked into sub-threads and only some closed, the report stays live as the entry point for whatever's still open.
- **Iterate on the latest while a thread is open.** Improve the highest-numbered live report rather than spawning a fresh one each round; a report stays live only while its questions are unresolved.
- **Batch sweep is for backlog, not the normal path.** If reports piled up un-archived, or a whole phase closes, sweep the spent cluster into `reports/archive/` in one move (ask first). Optionally merge a range into one `S<lo>-<hi>.html` baseline first — only if that pile is worth a single synthesis.
- **Reconciliation pass before folding a thread into the SOT.** Before distilling a multi-round thread's final conclusions, read every round in it side by side — not just the latest — and confirm the final edit set doesn't silently contradict, duplicate, or drop something an earlier round already settled. This is `consistency.md`'s single-source rule, applied to a report thread's own history.
- **Never delete; numbers never reused.** Archived reports stay verbatim in `archive/` for provenance.

## Index

- `reports/` is non-SOT, so its "what exists / what status" lives in a README index, kept in sync on every add/remove/archive (**consistency.md** Rule 1 gate). Rows are **lean** — filename · date · short status + one line; the reasoning lives in the report itself / in `decisions/`, not in the index.
- A report may be a file or a folder (per conventions); branches get subfolders.

## The three lifetime classes

Not restated here — see **`../ctx/SKILL.md § the model`**. `reports/` is **DISPOSABLE**. Hard constraints (index sync, disposable-not-scan-entry) live in **consistency.md** (Rule 1); this skill teaches the method.

---
name: consistency
description: The cross-cutting constraints every edit to a ctx knowledge base must satisfy — the four consistency rules (single source, same change / no drift, verify against canonical, the gate) plus the shared format conventions (numbering, archive-by-lifetime-class, file-or-folder units, README-per-folder). Each ctx skill loads this before it commits.
---

# ctx — the constraints every edit must follow

Every ctx skill loads this file before it commits. It has two parts:

- **Part 1 · the four cross-cutting rules** — what you must DO to keep the one truth true, across *any* activity (writing a spec, tracking progress, merging, reporting, scaffolding).
- **Part 2 · the shared format conventions** — how every folder numbers, archives, and indexes.

The **Verification gate** at the end is the executable checklist — run it over what you touched before any commit; the two parts below are what it checks.

The MUST/NEVER lines here are **binding hard constraints**, not advice — each skill loads and applies them before it commits. Both parts hold regardless of which skill is running. The lifetime classes (LIVING / APPEND-ONLY / DISPOSABLE) these rules assume are defined in [`../SKILL.md` § The model](../SKILL.md).

---

# Part 1 · The four cross-cutting rules

When the same fact lives in N docs (one says "4 reasons", another "5"), no reader — especially a future agent after a /compact — can tell which is canonical. The drift looks tiny but **destroys trust in the whole doc set**: "if they can't keep a number right, what else is wrong?" The same rot enters four ways across every activity: a fact copied instead of linked, a change that lands in code but not its doc, a count stated from memory, and a task declared done against no checklist. These four rules close those four doors.

> **Imperative restraint.** Reserve MUST/NEVER for genuine invariants — over-using aggressive imperatives dilutes the signal and over-triggers 2026 models.

## Rule 1 · Single source — one fact, one place (links, not copies)

- **MUST** keep every piece of knowledge in **one canonical location**; everything else **links** to it (Pragmatic Programmer Tip 15 — the documentation form of DRY). Identify the single canonical doc for each fact/enumerated set; others reference, canonical defines.
- **NEVER** copy a fact into a second doc. If two docs need it, one is canonical and the other links. "If you change one you'll forget the other — it's a question of *when*." *(Carve-out: a skill loads independently, so it MAY restate ONE load-bearing line and link back for the full version — see [`../SKILL.md` § Note on restatement](../SKILL.md).)*
- In the ctx model: current product truth lives only in `spec/`; "why" only in `decisions/`; work state only in `progress/`.

### One "current" per fact — supersede & stale-pointer hygiene

- **MUST** keep supersede chains readable **in place**: when content moves or a decision is replaced, the old location gets a one-line pointer to the new (for ADRs, the bidirectional status flip — mechanics in **ctx-spec**). The truth is the *chain*, not the latest file alone.
- **NEVER** let two documents both present themselves as "current truth" / "the thing to act on."
- **NEVER** leave a "go see X" pointer after X has been superseded by Y — repoint to Y (single current target; **no multi-hop stale chains**). Every forward reference resolves to a defined, current target in the same commit.

### Non-SOT indexes & handoffs (highest drift risk)

Handoffs / READMEs / summaries / reports-indexes drift most because they're written from memory, not sources.

- **MUST** generate them by reading the source docs, not chat history; re-verify every quantitative claim against canonical (Rule 3) before finalizing.
- **MUST** keep the `reports/README` (and any non-SOT folder index) listing exactly what exists, with status, updated on every add/remove/archive.
- **NEVER** trust a previous handoff's numbers as a source — re-verify against canonical.

### Disposable folders are not a scan entry

- **MUST NOT** use `scratch/` or `reports/` as a scan / entry point to "understand the project" — they hold rejected and obsolete ideas that surface as false positives. **Open a disposable file only when a SOT doc (spec / decision / progress) explicitly links to it.**

## Rule 2 · Same change, no drift — propagate in the same commit

Two flavors of the same discipline: cross-doc reference propagation, and code↔doc truth-sync. Both say: **the change that creates a divergence is the moment to close it. "Later" is how drift is born** — context is lost, the author moves on, the next reader inherits a lie.

### Cross-doc propagation (absorbs the folder-level reference-propagation convention)

- **MUST**, when a canonical name / term / count / number changes, grep **all** docs and update **every** reference **in the same commit**; keep a short term-mapping note so future sessions understand the migration. Never leave a dangling pointer or a stale count. This is the mechanism that lets LIVING docs be edited in place without drift.
- **NEVER** add/remove an enumerated item, or rename a term, without searching all cross-references first.

### Code ↔ doc truth-sync

A ctx knowledge base has two descriptions of one system: the **SOT** (`spec/` + `decisions/`) carries **intended** behavior; the **code** carries **actual** behavior. They must agree. *(The spec-side statement of this intent-vs-behavior authority model is **ctx-spec § Truth arbiter**; this rule enforces it on a behavior-changing commit.)*

- **MUST**, when a code change alters **observable** behavior, update its owning `spec/` section or `decisions/` ADR **in the same commit** (or the immediately following one). When they diverge, decide which is true: code is a bug → fix code to match spec; or the intent changed → update the spec/ADR (an in-place LIVING edit; a changed *choice* is a superseding ADR, not a rewrite).
- **NEVER** leave a divergence undecided — a spec describing behavior the code no longer has is a **silent bug** that misleads every future builder.

**What counts as a behavior change** (needs a doc-sync): trigger conditions/rules · command/API/IPC contract semantics · state transitions · storage/backend contract · anything that would change an EARS acceptance line. **Does NOT** (doc still true): internal refactor with identical external behavior · perf optimization that doesn't change contract · comment/variable renames. The test: *would the spec's acceptance criteria still pass, unchanged, against the new code?* Yes → no sync; No → the spec is now wrong, sync it.

- **MUST** run the **affected-docs sweep** before marking a behavior-changing task done: `grep -rE "<behavior keyword>" <ctx-root>/spec/ <ctx-root>/decisions/ <ctx-root>/design/ README.md`. Any hit still describing the old behavior is stale — fix it before "done". Do **NOT** sweep `scratch/` or `reports/` (DISPOSABLE — may hold superseded thinking on purpose).
- A detected-but-unfixed drift is an **open item** (note it as current position in `progress/` with a pointer to the offending spec section + code location), **NEVER** a standalone "drift log" doc — that would be a second source of truth about the same fact. Smallest sync that makes the SOT true again; do not rewrite a whole spec because one line drifted.

## Rule 3 · Verify against canonical — never from memory

- **MUST** verify any "N items / N reasons / N phases" matches the actual enumeration in the canonical doc **before commit** (`grep -rn "5 个" .` or the right pattern). Keep membership and naming identical across all references (same items, same order where order matters).
- **MUST** keep chapter/section numbering coherent (consecutive; one convention per doc); if a sequence skips, add an explicit note why; grep and update cross-references when renumbering.
- **NEVER** state a count from memory. **NEVER** write "about 5 / roughly N" in a technical doc — it signals you didn't check. **NEVER** orphan a referenced section number.

## Rule 4 · The gate — author the acceptance checklist BEFORE you execute

For any non-trivial task: **write an acceptance gate (a checklist) FIRST, execute, then validate against it — and do not report "done" until every item passes.** The checklist is authored as the *standard* for the work, not a post-hoc rationalization. This is the one pattern behind spec **acceptance-criteria**, progress **task-completion**, report **self-verification**, and the **ctx-folder DoD** below.

- **Standing gates** (per *class* of work, reused every time) live in the skills — this file's verification gate, ctx-spec's Tier-A GATE, the ctx-folder DoD below.
- **Ad-hoc gates** (per *this* task) live in `progress/` (the acceptance half of a task's Definition of Done) or `scratch/`.

### The ctx-folder Definition of Done (the gate for setup)

A ctx folder is *done* — set up correctly, not just non-empty — when ALL hold. (The folder *skeleton* itself is the model — see [`../SKILL.md` § The canonical folder](../SKILL.md); this checklist is the gate that verifies it.)

- [ ] **Lifetime skeleton exists:** `spec/ decisions/ progress/ reports/ scratch/` (plus `overview.md` / `README.md`).
- [ ] **Each has a README/index** — no folder is a bare pile.
- [ ] **SOT seeded:** `spec/` holds current truth (not empty stubs); `decisions/` holds ≥ the seed ADRs (the foundational choices already made).
- [ ] **Indexes in sync:** `reports/README` (and every non-SOT index) lists exactly what exists (Rule 1).
- [ ] **No orphan docs:** every doc is classified by lifetime — nothing sitting unclassified at the root.
- [ ] **Boundary decided:** in-repo `/ctx` vs external symlink chosen (see **ctx-adopt**), AND the confidentiality `.gitignore` is in place (a symlink-out or gitignored dir is verified never committed when the code repo is public).

---

# Part 2 · Shared format conventions

The format rules every folder in `ctx/` shares. Specialized folders (`reports/`, `spec/design/`) add to these; they never contradict them.

## Numbering

- **Zero-padded two digits: `01 02 … 99`**, extend to three only when a namespace is expected to pass 99. Zero-padding makes lexical order equal numeric order (`ls` sorts `01,02,…,10`; unpadded `1,10,2` does not). Consistent with ADR `NNNN` numbering.
- **Per-namespace, monotonic, never reused.** Each folder that numbers (`reports/`, per-subsystem specs, `decisions/`) has its OWN counter starting at `01`. No single global counter across folders — that is unmaintainable.
- **Archived numbers are not recycled.** A number consumed by an archived unit stays consumed; the next unit takes the next free number. (ADR numbering — 4-digit, sequential, never reused — is the canonical case; see ctx-spec.)
- **Every durable doc carries a number** except the singletons named by role (`progress.md`, `overview.md`, `services.md`, `README.md`, `DESIGN.md`).

## Archive mechanics — by lifetime class

The single most important convention: **how you retire a stale artifact depends on its lifetime class, not on a blanket rule.** A LIVING document NEVER carries a deprecated/stale section inline — that is false-positive noise a future agent wastes tokens on.

| Class | Retirement mechanic |
|---|---|
| **LIVING** — `spec/`, `spec/design/`, `overview.md`, `progress/` | **Localized change** (a section or two stale, the rest holds) → **edit in place**; git keeps the old text. **Foundational change** (the basis shifted, most needs rework) → move the WHOLE old doc to `<folder>/archive/` under the canonical archive filename (below) and write ONE clean new `<name>.md` reusing the canonical name — so the active doc reads like a single elegant final work, not a Frankenstein of old+new. Either way: **ripple-fix every internal reference in the same commit** (Rule 2), and record the *why we rewrote* as a **decision (ADR)** — the history lives there, keeping the spec clean. |
| **APPEND-ONLY** — `decisions/` (ADRs) | **Never edited, never deleted, never archived-by-status.** A reversed decision is superseded in place: flip the old one's status to `Superseded by NNNN`, add a bidirectional link, write a new ADR. The trail ("we thought X, then Y, which caused the change") IS the value — losing it means re-litigating settled decisions. The index carries the token cost, not a forced read. (Full status lifecycle + mechanics in ctx-spec.) |
| **DISPOSABLE** — `reports/`, `scratch/` | **Archive-aside.** Terminal artifacts nobody references as truth: on completion move the unit verbatim into the folder's `archive/`. Never delete by default. Disposable folders are **not a scan entry** — open a file here only when a SOT doc links to it (Rule 1). |

**Why the split**: a superseded *component* is dead weight (nobody needs the old button's CSS) → archive it. A superseded *decision* is live knowledge (it prevents repeating a mistake) → keep it, marked superseded. Same word "old", opposite handling — because their lifetime classes differ.

### Revise clean — the delta goes in the report/commit, not the doc

When you revise an authored doc after feedback (a report comment, a review, a new decision), the doc is **rewritten to read as clean current-truth** — NEVER accreted with inline-diff / tracked-change / `~~old~~ → new` marks or a stack of `EDIT:` / `UPDATE:` notes. Those turn a LIVING doc into a Frankenstein and re-introduce the stale-inline-section rot the archive rule exists to prevent. Instead:

- **The doc** stays one clean current-truth statement (LIVING localized edit), or a fresh clean version (foundational rewrite → archive the old verbatim first).
- **The delta** — what changed and why — is narrated in the **accompanying report / reply and the commit message**, not in the doc body. "What changed" → the report or `git log`; "what's true now" → the clean doc.

The old version is never lost — it lives where history belongs (git for a localized LIVING edit, an `archive/` move for a foundational rewrite or a DISPOSABLE report), not smeared through the current doc.

Archive folders are **per-folder, plain name `archive/`** (no `_`/`.` prefix — `.archive` hides from ripgrep; `_archive` reads as "don't build" to static-site tools and doesn't sort to top).

**Archive filename (single canonical convention — stated ONCE here, everywhere else links back):** an archived unit keeps its original stem and takes a date suffix at **day precision** — `<stem>.<YYYY-MM-DD>.md` (or `<stem>.<YYYY-MM-DD>/` for a folder unit). A **milestone label is an OPTIONAL prefix segment** when it aids scanning: `<stem>-<milestone>.<YYYY-MM-DD>.md`. Day precision (not month) so multiple archives of the same stem in one month stay distinct and lexically ordered. No skill restates this format string — `ctx-progress`, `ctx-spec`, and `spec/design/` all point here.

## Numbered units: file OR folder

A numbered unit may be a single file (`06.html`, `06.md`) or a folder (`06/` with `06/index.html` + parts). Default to a single file — most units are one self-contained thing. Use a folder only when a unit genuinely has sub-parts or branches. **Archiving moves the whole unit** (the file, or the folder entire — never split it).

## Per-folder README

Every non-SOT folder (`reports/`, `scratch/`) carries a `README.md` stating what its files are for and — for `reports/` — an index of what exists and its status. This is how a non-SOT area stays navigable without being deep-scanned. (`scratch/` needs no index; nobody browses it.)

---

# Verification gate

Before finalizing ANY multi-doc edit, any behavior-changing code commit, or any edit that touches an index / counted set / supersede chain: run the four rules over what you touched.

```
verify(edit) → pass | fixes

# RULE 1 · SINGLE SOURCE
assert no_duplicated_fact(docs)                            # each fact ONE canonical home; others link
assert one_current_per_fact(docs) ∧ supersede_links_bidirectional(docs)
for ptr in grep(docs, r"see |见 |→|§|ADR-|Q[0-9]"): assert resolves_to_current_target(ptr)  # no dangling / stale / multi-hop
assert index_matches_dir("reports/") ∧ index_matches_dir(non_sot_folders)
assert disposable_not_used_as_scan_entry(scratch, reports)

# RULE 2 · SAME CHANGE, NO DRIFT
for term in renamed_or_recounted: assert grep(docs, term).all_updated()   # same-commit propagation
if code_change_alters_observable_behavior:
    assert owning_spec_or_decision_updated_same_commit()  # code↔doc truth-sync
    assert swept(SOT, changed_behavior_keyword)           # affected-docs sweep before "done"

# RULE 3 · VERIFY AGAINST CANONICAL
for claim in grep(docs, r"[0-9]+ (个|items|reasons|phases|steps)"):
    assert claim.n == count(canonical_enumeration(claim))  # never from memory
assert naming_and_numbering_match_canonical(docs)

# RULE 4 · THE GATE
assert acceptance_checklist_authored_before_execution(task)
assert all_checklist_items_pass(task)                     # never report "done" otherwise
if setting_up_a_ctx_folder: assert ctx_folder_DoD_passes()

fail any → fix the gap, re-verify. Never leave it.
```

**Fail any assertion → do not commit; fix the underlying violation first, then re-verify.** A drop, a stale pointer, a restated fact, a code↔doc divergence, or an unchecked count is never "cleaned up later" — later never comes.

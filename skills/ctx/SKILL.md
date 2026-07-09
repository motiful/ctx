---
name: ctx
description: Sets up and orchestrates a ctx knowledge base — a lean, living single source of truth for multi-session AI-agent work — and classifies any doc or finding by its lifetime, routing the work to the right companion skill. Use when starting or scaffolding ctx on a new project, when unsure where a finding belongs (spec vs decision vs progress vs scratch), when unsure which ctx skill handles a task, or when docs have sprawled into redundant, contradictory piles that need converging. Entry point for the collection; routes to ctx-adopt (existing repos), ctx-merge (converge sources), ctx-spec (specs and ADRs), ctx-progress (work tracking), ctx-report (reports for a human), ctx-serve (host processes across sessions), ctx-compact (pre-reset checkpoint). For an existing/brownfield repo, use ctx-adopt instead.
license: MIT
metadata:
  author: motiful
  version: "2.3"
---

# ctx — a living, spec-centered single source of truth

ctx sets up and runs a knowledge base that stays a lean, living single source of truth across many agent sessions. Its own job is small and load-bearing — **scaffold** the folder, **classify** every finding by lifetime, **route** the domain work to a companion, and run the **gate**; the heavy domain work lives in the companions. This is the entry point: read the model (classify depends on it), then run the procedure and route.

## The model — two ideas

### 1. Organize by LIFETIME, not stage

Three lifetime classes. The archive rule and the edit rule both follow the class — that is the whole model. **This table is the single source for the lifetime classes; companions reference it, they do not redefine it.**

| Class | What | Iron law | Who belongs |
|---|---|---|---|
| **LIVING** | current truth | edited in place; always current; never carries a stale/deprecated section inline | `spec/` (product truth, incl. `spec/design/`) + `overview.md` + `progress/` (work truth) + `services.md` (infra topology, when present — see ctx-serve) |
| **APPEND-ONLY** | history | never edit or delete an accepted entry; supersede in place | `decisions/` (why; ADRs) |
| **DISPOSABLE** | raw / process | non-authoritative; sink the confirmed, archive-aside the rest | `scratch/` (ALL raw: notes, prompts, research dumps, comparisons) + `reports/` (HTML a human reads for review) |

### 2. The SOT is GENERATIVE

**`spec/` + `decisions/` are the single source of truth, and they are generative.** Any explanation, report, tutorial, or option-comparison is a **short-term, on-demand derivation FROM the SOT** — produced when you need to communicate with someone, then discarded. **There is no long-term "explanation" document.** If a spec states the constraint and an ADR records the choice + its real alternatives + consequences, then "why did we do X, not Y" is *derivable on demand* — so you do not store it. This is what keeps the base lean.

- **No standalone `log`.** "What happened" = git history. "Why we abandoned X" = a decision. Don't keep a separate event log.
- **Two living docs is fine** (no SSOT violation): `spec/` = the PRODUCT (what it is / how it works), `progress/` = the WORK (where we are / what's next). Different domains. Guard: `progress/` **references** spec/decisions by pointer, never restates a product fact.
- **Archive follows lifetime class** (mechanics AND the canonical archive filename in `references/consistency.md`): LIVING → fix in place, or on a *foundational* rewrite move the whole old doc to `<folder>/archive/` under the canonical archive filename and write ONE clean new doc; APPEND-ONLY decisions → never archived, supersede in place (the trail is the value); DISPOSABLE → archive-aside.

## What goes where (classify by lifetime — write from any direction, NOT a waterfall)

Order does not matter. Whatever you produce, classify it:

1. **Current product truth** → `spec/` or `overview.md`, edit in place.
2. **A *made* choice + why + the real alternatives** → append `decisions/NNNN`, supersede-don't-rewrite. *(Only decided things — see ctx-spec for the ADR content boundary. Undecided deliberations do NOT go here.)*
3. **Where we are / what's next** → `progress/`, edit in place, pointers only *(see ctx-progress)*.
4. **Raw / uncertain: notes, prompts, research dumps, comparisons** → `scratch/`, non-authoritative.
5. **Thinking laid out for a human to review** → `reports/` (disposable HTML; see the **ctx-report** skill).

## Execution Procedure

```
build_or_maintain_kb(project_or_pile) → living_kb

# STEP 0 — Load the cross-cutting constraints (four rules + format conventions)
read("references/consistency.md")        # single-source · same-change · verify-canonical · gate + numbering/archive-by-class
ensure ctx/ skeleton exists — scaffold from assets/ctx-skeleton/ (progress/ · spec/ · decisions/ · reports/ · scratch/ · README.md)

# STEP 1 — Classify by LIFETIME, not stage (write from any direction; NOT a waterfall)
for each finding / note / doc:
    cls = classify(finding)
        # current product truth         → spec/ or overview.md   (edit in place)
        # a MADE choice + why + rejected → decisions/NNNN         (append; supersede, never rewrite)
        # where we are / what next       → progress/              (edit in place; pointers only)
        # raw / uncertain / research      → scratch/               (non-authoritative; not a scan entry)
        # normative extract of a theory   → spec/ + decisions/     (concept can be normative)

# STEP 2 — Converge scattered sources into the KB (any consolidation of ≥2 sources)
if consolidating notes / reports / subagent outputs:
    Skill("ctx-merge", sources)          # disposition ledger; a drop is a recorded decision, never silent

# STEP 3 — Write build-grade docs
if writing / restructuring a spec, ADR, or design doc:
    Skill("ctx-spec", target)            # formats, granularity, ADR boundary + numbering, spec/design/, the writing standard

# STEP 4 — Track progress / hand off between sessions
if updating work state, opening a subtree, or handing off to a new session:
    Skill("ctx-progress", ctx/progress/) # single-file-or-tree, frontmatter, handoff, archive-when-long

# STEP 5 — Producing / distilling / merging a report for human review?
if writing or distilling an HTML report for the human:
    Skill("ctx-report", ctx/reports/)        # report format · distillation · rolling-merge · archive

# STEP 6 — Apply the cross-cutting constraints (mandatory before finishing any multi-doc edit or behavior-changing commit)
apply("references/consistency.md")       # verify single-source · same-change (incl. code↔doc) · verify-against-canonical · the gate — fix any violation before committing

# STEP 7 — Stop (JBGE): smallest living-doc set that lets an agent build correctly. Do not over-document.
```

Each `read()` and `Skill()` above is a decision-layer entry — enter the module when the step runs; do not paraphrase it here.

## When to call the companions

**You (the agent) classify and route; the user never hand-picks a companion.** The user describes their work in plain language ("nail down the design for X, then build it" / "merge these notes" / "where does this finding go?"). Read the intent and route by what it needs. When one ask spans **both** a durable contract and transient work — the common *"settle the docs, then execute"* case — produce **BOTH**: the durable "what must be true" → `spec/` (via **ctx-spec**) and the doing/tracking → `progress/` (via **ctx-progress**), linked by `REQ-### ↔ T###`. Never ask the user to choose "spec or progress" — that classification is this skill's job. Signals: *should/must/design/architecture/requirements* → durable → spec; *let's do/plan/next/where are we* → transient → progress; *a pile of notes to settle* → **ctx-merge**.

- **Converging scattered notes / many reports / subagent outputs into the KB** → **ctx-merge** (merge discipline + human-adjudicated routing). Use it for any consolidation of ≥2 sources — a silent merge drops content.
- **Writing or restructuring a spec, ADR, architecture doc, or design/design-system doc** → **ctx-spec** (formats, templates, granularity, ADR content boundary; `spec/design/` and the writing standard live here).
- **Updating work state, opening a progress subtree, or handing off to a new session** → **ctx-progress** (single-file-or-tree progress, frontmatter, handoff protocol, archive-when-long).
- **Writing an HTML report for the human to review, distilling their comments, or merging a pile of reports** → **ctx-report** (report format, keep/drop/unsure distillation, rolling-merge, archive-aside).
- **Bringing an existing (brownfield) repo under ctx — where `/ctx` mounts, in-repo vs external symlink, onboarding a messy tree** → **ctx-adopt** (minimal-disruption onboarding; routes the actual doc-writing to the domain skills).

The two companions below are not document skills — one is **operational**, one is **meta** (it orchestrates the others):

- **Hosting a dev server / watcher / build daemon that must survive a reset or be shared across parallel sessions** → **ctx-serve** (tmux-hosted processes, a committed `services.md` topology manifest, live status detected-not-stored).
- **Preparing to `/compact` or reset the session — sweeping everything decided into its SOT home first** → **ctx-compact** (a thin cross-SOT checkpoint over ctx-progress + decisions/spec + the folder indexes; introduces no new rule).

The cross-cutting hard constraints — single-source · same-change (incl. code↔doc) · verify-against-canonical · the gate — are **not a companion you route to**; they live in **[`references/consistency.md`](references/consistency.md)**, which every skill (and STEP 0/6 above) loads before it commits.

---

*The rest of this file is orientation — the problem ctx solves, the finer classification calls, the folder shape, and honest limits. The procedure above is the operative part; read on when you need the why.*

## The problem this solves

Long, multi-session AI-agent research/design produces documents that **rot** into a pile that is **redundant, contradictory, and stale**. Symptoms: docs multiply endlessly; changing one thing forces chasing a chain of edits across many docs; old docs go 50%-true/50%-false so you can't tell what's current; the process order feels chaotic ("research first or design first?"); dozens of review reports pile up and merging their conclusions becomes its own unsolved problem.

Root cause: organizing docs by **pipeline stage** (requirements→research→design→decisions→progress) leaves a persistent, editable, go-stale doc at every stage.

## Theory & evidence — not a separate class

Psychology, research evidence, a chosen theoretical framework: **conceptual content can absolutely be normative** (normative = "is this binding on what we build", not "is it concrete vs abstract"). So split it:

- The **normative extract** — which paradigm, how it's applied, and (in `decisions/`) why this one not others — is BINDING → goes to `spec/` (a design principle) + `decisions/` (the framework choice).
- The **raw literature / evidence body** → `scratch/` (disposable), cited by the spec/decision.
- **Classic theory** → just name it (the model already knows it). **New (2026) theory** → cite the URL + a one-line summary in the spec. **A theory you designed yourself** IS a spec (you authored it).

Reliability flows from the SOT's reference: a disposable file earns trust *only* where a SOT doc links to it.

## Disposable is NOT a scan entry (false-positive guard)

`scratch/` and `reports/` are non-SOT: they are not a scan / entry point. A disposable file is opened *only* where a SOT doc (spec / decision / progress) explicitly links to it — deep-scanning `scratch/` to "understand the project" surfaces rejected and obsolete ideas as false positives, as if they were current truth. (This is a model characterization; its binding MUST/NEVER form is **[`references/consistency.md`](references/consistency.md) Rule 1**.)

## Normative vs informative — spec vs report

The split above is a **status** distinction: a **spec** states conclusions and constraints (normative, build-grade); a **report** is discursive and disposable (informative). A spec that teaches, compares options, or narrates what happened has let informative content leak in — move it to a report (for review) or an ADR (the why). The one "why" that *stays* in a spec is its **INTENT** (a one-clause *so-that* that scopes a rule). The full writing standard — the four axes (normative/informative · RFC-2119 · IRB why-discipline · granularity) — lives in **ctx-spec**.

## The canonical folder

The knowledge root is `ctx/` — a single folder at the project root. This is the stable **mount point**; its backend is an in-repo directory by default, or a gitignored symlink to an external store when the context must not ship (see **ctx-adopt**).

```
ctx/                       # knowledge root — the single SOT for product + work truth
├─ progress/               # LIVING · work truth: current focus + next + "read first"
│                          #   (single file progress.md when small; a tree when it grows — see ctx-progress)
├─ overview.md             # LIVING · product truth, system level (C4 Context + Container)
├─ services.md             # LIVING · infra topology: long-running processes this project hosts (optional; see ctx-serve)
├─ spec/                   # LIVING · product truth (see ctx-spec)
│  ├─ <subsystem>.md       #   one clean current-truth doc per subsystem
│  ├─ design/              #   design is a spec SUBTYPE; its .html/.json ARE the SOT (ctx-spec → design-system.md)
│  └─ archive/             #   whole old specs on a foundational rewrite (versioned name)
├─ decisions/NNNN-*.md     # APPEND-ONLY · why (ADR; see ctx-spec). Superseded → status flip in place, never deleted
├─ reports/                # DISPOSABLE · HTML a human reads for review (NON-SOT; see the ctx-report skill)
│  └─ archive/
├─ scratch/                # DISPOSABLE · ALL raw: notes, prompts, research dumps, comparisons (NON-SOT, not a scan entry)
│  └─ archive/
└─ README.md               # index (esp. reports/) + the project's chosen formats
```

*(No separate `CONVENTIONS.md` file — the project's chosen formats live in `ctx/README.md`, so it can't be confused with this skill's `references/consistency.md`.)*

### Definition of Done for a ctx folder

The skeleton above is the *model* — what a correct ctx folder looks like. Whether a given setup actually *is* done (skeleton present, each folder indexed, SOT seeded, indexes in sync, no orphan docs, backend + confidentiality boundary decided) is checked against the **ctx-folder Definition-of-Done checklist**, which lives with the other standing gates in **[`references/consistency.md`](references/consistency.md) § Rule 4 (the gate)**. Model here; the DoD gate that verifies it there.

## The external store — how a shippable artifact records its own making

When the deliverable is a **shippable artifact** (a skill, an app, a video) whose docs must NOT ship with it, keep the docs in a sibling **`<name>-ctx/`** (external, never shipped), managed by *this* methodology. **That `<name>-ctx/` folder itself IS the knowledge root** — it plays the role of `ctx/`, so its children are `spec/ decisions/ progress/ reports/ scratch/` **directly** (no nested `ctx/`). The published artifact = the output; the external store = *why it is the way it is* (its spec + its ADRs). This is where a design decision like "we **supersede** old ADRs in place (never archive them), because a superseded decision is live knowledge" is recorded — so the reasoning is never lost. **This collection dogfoods it exactly: this skill ships as `ctx/`, its own making lives in the sibling `ctx-ctx/`.**

## Cross-cutting rules & the read-side habit (pointers, not mechanics)

The prescriptive cross-cutting MUSTs — **single-source · same-change (incl. code↔doc) · verify-against-canonical · the gate** — plus the shared format conventions live in **[`references/consistency.md`](references/consistency.md)**, which every skill loads before it commits. In particular the **gate** (author the acceptance checklist *before* you execute; validate before you claim "done") is **consistency.md Rule 4** — `ctx` names the pattern behind spec acceptance-criteria / progress task-completion / report self-verification / the ctx-folder DoD; consistency.md holds the mechanics.

**Read-side habit** (why the append-only trail earns its keep): before an architecturally-significant choice, the `decisions/` trail + reject log get scanned so a settled question isn't re-litigated and a rejected concept isn't reintroduced — a decision nobody consults is just history. Its binding form is **ctx-spec § Hard constraints** (ADR discipline); the reject log feeds from **ctx-merge**.

## The loop

```
CHAOS (notes/conflicts) → debate/converge → scattered good questions + local answers
   → [ctx-merge] sink into the KB → spec/decisions (current truth + why)
   → [ctx-spec] write build-grade specs → agent builds → tests are the arbiter of behavior
   → new conclusions fold back into the KB (edit spec in place / append decision). Repeat.
```

## Stop condition (don't over-build docs)

The goal is the SMALLEST set of living docs that lets an agent build correctly. Apply JBGE (Just Barely Good Enough): stop documenting when the next sentence costs more than its value. Anything derivable from code/tests (API refs, generated docs) is **generated, never hand-maintained**. Anything derivable from spec + decisions (explanations, comparisons) is **derived on demand, never stored**. Scratch and reports are disposable by default.

## Honest limits

Every mechanism trades **discipline** for cleanliness: append-only decisions accumulate (but appending is cheap and the index gates the token cost); living specs rot without a freshness check; scratch/reports pile up if never swept; confidence labels become theater without periodic re-verification. The model removes the *structural* causes of rot — it does not remove the need to actually maintain.

## References

- `references/consistency.md` — the cross-cutting constraints every edit must satisfy: the four rules (single-source · same-change incl. code↔doc · verify-against-canonical · the gate) + the shared format conventions (numbering, archive-by-lifetime-class, file-or-folder units, README-per-folder). Every skill loads it before committing.
- *(The `reports/` workspace method moved to the **ctx-report** companion skill — no longer a reference here.)*

> **Note on restatement (DRY carve-out):** independently-loaded skills and references MAY restate a load-bearing rule in one line and link back here for the full version. Within *this* skill (SKILL.md + its own `references/`), each fact has exactly one home.

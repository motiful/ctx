---
name: ctx-spec
description: Writes specs, ADRs, architecture and design docs that an AI coding agent can build directly from — the right granularity (constraints, not hand-written implementation), testable EARS acceptance criteria, and ADR discipline (record only decided choices, supersede in place). Use when writing or restructuring a spec, a decision record (ADR), an architecture doc, or a design doc in a ctx knowledge base, or when deciding how detailed a spec needs to be.
license: MIT
metadata:
  author: motiful
  version: "1.5"
---

# ctx-spec — Write Docs an Agent Can Build From

> The lifetime model and shared conventions live in [`../ctx`](../ctx/SKILL.md); this is the spec / ADR / design how-to layer.

## How a spec is written — the writing standard

A spec is an **authored doc an agent reads once and builds from**. Four orthogonal axes govern every line; each governs a different thing, so they compose without clashing:

1. **Status — normative vs informative.** A spec states **conclusions and constraints** (normative). A note, example, or figure is **never** a requirement (informative). State a binding claim as a MUST/SHALL line — don't let a diagram imply it.
2. **Strength — RFC 2119.** `MUST / SHOULD / MAY`, all-caps, grep-able (all-caps only per [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)). Lower-case "must" is prose, not a requirement.
3. **Why-discipline (IRB) — which "why" may stay.** Classify every "why" by what it does for the agent that reads the spec:
   - **INTENT** — a one-clause *so-that* that scopes a rule (`X, so that Y`) so the agent generalizes to cases the spec never enumerated. **Stays inline** with the rule.
   - **RATIONALE** — the justification of a choice among alternatives (`chose X over Y because Z`). **Goes to an ADR** (`Considered options` + `Decision outcome`), never the spec body.
   - **BACKGROUND** — context the model already knows, or history that changes no action. **Cut** it.
   - **Delete-test:** delete the clause — does an edge-case action change? *Generalizes the rule* → INTENT (keep). *Only justifies a past choice* → RATIONALE (→ ADR). *Nothing* → BACKGROUND (cut).
4. **Granularity** — specify the **constraint, not the mechanism**; testable (EARS), versioned. Its own section below (the most important one).

**The one anti-pattern (mode bleed):** a spec that teaches, compares options, or narrates what happened has let Explanation leak into a normative doc. Rationale → an ADR; comparison / review thinking → a report. The single "why" that *belongs* in a spec is **INTENT** — it carries normative force, because deleting it changes conformance at the edges.

## Execution Procedure

```
write_spec_or_record(target) → build_grade_doc

kind = classify(target)   # subsystem-spec | decision(ADR) | design | architecture(HLD) | theory

if kind == "theory":                        # psychology / research-evidence / a chosen framework
    split(target): normative_extract → spec (principle) + decisions (why-this-framework) ; raw_body → scratch/

if kind == "subsystem-spec":
    apply(granularity_rule)                 # specify constraint not mechanism; no hand-written LLD
    fill(spec_template)                     # Tier A precise (EARS + interfaces + schemas + invariants) · Tier B loose (HLD sketch)
    mark_open_questions()                   # [NEEDS CLARIFICATION] while draft

if kind == "decision":
    assert decided(target) ∧ significant(target)   # a MADE, significant choice — never a "proposed" parking lot for open options
    n = next_adr_number()                   # NNNN, 4-digit, sequential, NEVER reused
    write(f"decisions/{n}-title.md", adr_template)   # rich enough that "why X not Y" is derivable later
    if reverses(prev): set(prev.status, f"Superseded by {n}"); link_bidirectional(prev, n)   # supersede in place, never delete
    update("decisions/README.md")           # index row: number · title · status · date

if kind == "design":
    read("references/design-system.md")     # flat-first structure · layer taxonomy · DTCG tokens · motion · content boundary · tool-free standard

if kind == "architecture":
    place_hld()                             # C4 L1+L2 → overview.md ; L3 → per-subsystem Tier B ; L4 → agent generates

assert tierA_is_testable ∧ out_of_scope_stated ∧ no_hand_written_LLD    # GATE
apply("../ctx/references/consistency.md")                               # cross-cutting rules + conventions — verify before commit
```

## The granularity rule (the most important thing)

**Specify the constraint, not the mechanism. Loosen the prose, harden the tests.**

- Spec to: verifiable acceptance criteria (EARS / Given-When-Then) + named interfaces + data schemas + invariants + security constraints + explicit out-of-scope. Stop there.
- Do NOT hand-write low-level design (class/method internals) — the agent generates that in code. Big-company BRD→PRD→HLD→**LLD** collapses: BRD+PRD fold into one spec, HLD survives, LLD disappears as a human doc.
- Under-specify → the agent drifts/invents intent. Over-specify → you've hand-written pseudo-code (waterfall, brittle). Sweet spot: constraints + acceptance criteria; the agent picks the how. ("session lookup must be O(1)", not "use a HashMap".)
- Scale-dependent: loose specs for prototypes/recoverable work; hardened contracts for transactional/regulated systems.
- **Traceable end-to-end.** A user story threads forward through its requirement → design → task and stays followable (`story → REQ-### → design → T###`, where the `T###` task rows live in `progress/` and point back to a `REQ-###`); IDs are immutable and never reused. A criterion nobody can trace to a task won't get built; a task with no requirement is scope creep.

| MUST be machine-precise | SHOULD stay loose (agent decides) |
|---|---|
| interfaces / API contracts / signatures | internal implementation approach |
| data schemas + types | internal class/module structure |
| acceptance criteria (EARS/GWT, binary pass/fail) | algorithm choice where any correct one works |
| invariants & business rules | file organization, private naming |
| security / auth / privacy / compliance | micro-optimizations |
| error & edge taxonomy; non-functional limits | |
| out-of-scope / non-goals (agents can't infer from omission) | |

## spec.md template (per subsystem)

```markdown
# <subsystem> Spec
status: draft|active · owner: <name> · last-verified: <YYYY-MM-DD>

## 1. Intent & why            [MUST] problem + who + why now
## 2. Non-goals               [MUST] what it explicitly does NOT do
## 3. User scenarios          [MUST] As <role> I want <X> so that <Y> — P1/P2/P3

— TIER A · INTENT/BEHAVIOR (precise, verifiable) —
## 4. Requirements            [MUST] one row per requirement, each with an immutable `REQ-###` id
                              e.g. `REQ-001` <the requirement> — traces from scenario §3
## 5. Acceptance criteria     [MUST] EARS: WHEN <trigger> the system SHALL <response>; one clause = one test
                              each criterion cites the `REQ-###` it verifies
## 6. Interfaces & contracts  [MUST] public signatures, endpoints, event schemas
## 7. Data models             [MUST if data] entities, field types/constraints
## 8. Invariants & rules       [MUST if domain logic]
## 9. Constraints             [MUST] security/auth/privacy + non-functional limits
## 10. Error & edge behavior  [MUST] failure taxonomy, retry/fallback

— TIER B · CURRENT STRUCTURE (descriptive, deliberately loose) —
## 11. Architecture sketch    [OPT] HLD only — components + connections (Mermaid). NOT class-level
## 12. Existing code map       [OPT] where it lives / files to touch
## 13. Implementation notes    [OPT] hints; agent may override with rationale

## 14. Open questions [NEEDS CLARIFICATION]  [MUST while draft]
## 15. Assumptions            [OPT]

# Tasks are NOT part of the spec. The ordered T### task list + status live in progress/
# (ctx-progress); each T### points back to the REQ-### here that it implements.
```

The `REQ-###` (§4) fields make each requirement **individually addressable**. The traceability chain `story (§3) → REQ-### (§4) → acceptance criterion (§5) / design (Tier B) → T### (in progress/)` is followable end-to-end — the spec owns story→design; the `T###` task rows live in **`progress/`**, each pointing back to the `REQ-###` here that it implements. IDs are immutable and never reused (granularity rule above; a hard constraint — see § Hard constraints below).

> **Tasks live in progress, not the spec.** The spec ends at requirements + design (§1–§13, §14–§15). The ordered task list — the `T###` rows, their status (done / active / blocked), and the working construction plan — lives in **`progress/`** (ctx-progress); each `T###` points back to the `REQ-###` it implements. Rule of thumb: **spec defines what must be true; progress tracks the doing.** This keeps the durable contract (spec, never archived while the product lives) cleanly separate from the transient plan (progress, archived when done). A one-off chore with no durable product-truth delta gets no spec at all — the whole plan is just `progress/`.

**EARS keywords:** Ubiquitous `the system SHALL …`; Event `WHEN <trigger> …`; State `WHILE <state> …`; Unwanted `IF <cond> THEN … SHALL …`; Optional `WHERE <feature> …`. Review Tier A like a contract (precise/testable); Tier B like a map (orient, not binding). If you're writing Tier B to method level, move that constraint UP into Tier A as an acceptance criterion or interface.

## Truth arbiter

- **Intent** (what should be) → the **spec** is authoritative; if spec says X and code does Y, the code is wrong.
- **Behavior** (what actually is) → the **running code + tests** are authoritative; a doc that disagrees is stale.
- Bridge: compile EARS/GWT acceptance criteria into CI tests so the two stay aligned.
- **Recovering from drift** — when the agent builds the wrong thing, fix the **spec first**, then re-run; never patch code past a stale spec. The spec is the durable artifact, code is its current expression; treat the spec as a **living doc, not a one-shot prompt**. (Stale specs are the #1 spec-driven-dev failure — Kiro / Spec-Kit; updating the spec first is how you keep it alive.)
- When the drift is **code-vs-doc** (behavior changed but the spec didn't) rather than agent-vs-spec, keeping the two in truth-sync in the *same change* is **[`../ctx/references/consistency.md`](../ctx/references/consistency.md) Rule 2 (same change, no drift)** — this section states the authority model; that rule enforces it on a behavior-changing commit.

## Theory & evidence — conceptual content CAN be normative

A theory / psychology model / research-evidence input is not disposable by default (normative = "binding on what we build", not "concrete vs abstract"). **Route it by splitting:**

- **Normative extract → the SOT.** *Which* paradigm and *how* it's applied → a **spec** design principle. *Why* this framework, not others → an **ADR**.
- **Raw literature / evidence body → `scratch/`** (disposable), cited by the spec/decision. Reliability flows from the SOT link, not from the file sitting there.
- **Classic theory** → just name it (the model already knows it). **New (2026) theory** → cite the URL + a one-line summary in the spec. **A theory you designed yourself IS a spec** (you authored it — write it as a principle).

## ADR (decision record) format, numbering, and how it's read

```markdown
# NNNN. <short noun-phrase title>
* Status: proposed | accepted | deprecated | superseded by NNNN     — see status lifecycle below
* Date: YYYY-MM-DD · Deciders: <names>
## Context and problem statement   — the forces in tension
## Decision drivers       (optional)
## Considered options              — real alternatives (not straw men)
## Decision outcome                — chosen option, because …
### Consequences                   — good AND bad
## Confirmation           (optional) — how it's enforced (review / test / lint)
```

### What belongs IN an ADR (the content boundary)

An ADR records **a decision that was actually MADE**, rich enough that **"why X, not Y" is DERIVABLE from it later** — this derivability is *exactly* why the collection stores no long-term explanation docs. It holds:

- the **decision** made + its **real considered alternatives** (not straw men),
- the **outcome** (chosen because…) + **consequences** (good AND bad), + optional confirmation.

**What stays OUT:**

| Not an ADR | Goes to |
|---|---|
| **Undecided deliberation** (options still open) | `progress/` or the live session — NEVER a "proposed" parking-lot ADR |
| **Mutable current-state fact** | `spec/` |
| **Implementation detail** | code / `spec/` |
| **An event log** ("what happened") | git history |

**Significance bar:** write an ADR when a decision is **architecturally significant / costly-to-reverse / non-obvious / would-otherwise-be-relitigated**. Trivial reversible choices get **no ADR** — they're noise.

**Rejected-alternative clause (soft norm).** When a choice meaningfully turned *away* from an option, capture it — even a one-line decision-log entry should read `— rejected X because Y` where relevant. This feeds the reject-log that the read-side scan consults, so the same option isn't reintroduced later. (In a full ADR this is already carried by `Considered options` + `Decision outcome`; the clause matters most for the lightweight one-liners.)

**Before deciding — read the trail (read-side scan).** Before an architecturally-significant choice, SCAN `decisions/` + the reject-log so you don't re-litigate a settled question or reintroduce a rejected concept. This is the write-side payoff of the append-only trail — a global habit (see `../ctx/SKILL.md § the read-side habit`; a MUST in § Hard constraints below).

### Status lifecycle (precise)

| Status | Meaning |
|---|---|
| `proposed` | **Transient**, awaiting acceptance. NOT a place to park undecided options — a proposed ADR is a decision about to land, not an open debate. |
| `accepted` | Current; the decision is in force. |
| `deprecated` | The decision was **retired and NOTHING replaces it** — it no longer applies. (Distinct from superseded.) |
| `superseded by NNNN` | **Replaced by a specific new ADR.** Bidirectional link; old body left untouched. |

**Live-log tag mapping.** A real project often tracks decisions inline with lightweight tags before they become formal ADRs. Map them: `[LOCKED]` = `accepted`; `[DRAFT]` = `proposed` (a decision about to land, not an open debate); `[OPEN]` = **not yet a decision at all** → it belongs in `progress/` (or a spec `[NEEDS CLARIFICATION]` block), NOT in `decisions/`. Only `[LOCKED]`/`[DRAFT]` graduate to an ADR; `[OPEN]` never does until it's decided.

- **One decision per file** — `decisions/NNNN-title-with-dashes.md`. NOT many-in-one-file: per-file numbering, per-file status, and per-file supersede links all assume it, and an agent can open exactly the one it needs (cheaper than loading a monolith).
- **Numbering:** sequential, 4-digit zero-padded (`0001`), monotonic, **never reused** — Nygard's original rule.
- **Immutability: a decided ADR is NEVER rewritten.** You only ever APPEND a supersede pointer and flip the old status — the original clause text stays exactly as written. Reversing a decision = supersede in place, NEVER edit-the-old / delete. Write a NEW ADR with `Supersedes: NNNN`; flip the old to `Superseded by NNNN` (bidirectional). The old file stays, body untouched — the truth is the whole chain. This is a strong, documented CONVENTION (Nygard, Fowler, AWS, Microsoft, adr-tools), not an ad-hoc choice; the append-only trail ("we thought X, then Y") is exactly what prevents re-litigating settled decisions.
- **The index carries the token cost, not a forced read.** `decisions/README.md` is a table (number · title · status · date · supersedes/superseded-by), generatable by adr-tools/adr-log. An agent reads the index first, reads `spec/` for "what to build", and opens a specific ADR only for "why is this like this / can I change it". A superseded ADR costs one index row, not a full read — so keeping history is cheap.
- ADRs are **never archived** (unlike specs) — see archive rule below.

### Worked example — the supersede chain (bidirectional, in place)

`decisions/0003-session-store.md` (flipped when 0007 lands — body below the header untouched):

```markdown
# 0003. Session store in Postgres
* Status: Superseded by 0007
* Date: 2026-02-10
Chose Postgres for session rows: one datastore, transactional with the user table.
```

`decisions/0007-session-store-redis.md`:

```markdown
# 0007. Move session store to Redis
* Status: accepted · Supersedes: 0003
* Date: 2026-06-30
p99 session lookup missed the O(1) target under load; moved to Redis. Trade-off: a second datastore to operate.
```

The chain (0003 ⇄ 0007) reads in place — no deletion, no archive; the index carries both rows.

### Worked example — minimal spec skeleton (Tier A vs Tier B headers)

```markdown
# Auth Spec — status: draft · last-verified: 2026-06-30
## 4. Requirements          [Tier A] REQ-001 an expired token must be rejected
## 5. Acceptance criteria   [Tier A] WHEN a token expires the system SHALL return 401 (verifies REQ-001)  ← one clause = one test
## 6. Interfaces            [Tier A] POST /session → {token, exp}
## 11. Architecture sketch  [Tier B] gateway → auth-svc → session store (Mermaid, not class-level)
# the T001 task (implement token-expiry check → REQ-001) lives in progress/, NOT the spec
```

## HLD / architecture (C4) — placement

- **System level (C4 Context + Container)** → ONE top-level `ctx/overview.md`. Whole-system shape, tech choices, inter-service comms. Global, slow-changing.
- **Subsystem level (C4 Component)** → inside each `spec/<subsystem>.md` Tier B. Co-evolves with its spec.
- **Code level (C4 L4)** → not drawn; the agent generates it.
- Diagrams: Mermaid-as-code (`flowchart` / `sequenceDiagram` / `erDiagram` render natively on GitHub; C4 Mermaid blocks do NOT render on GitHub — use PlantUML+C4-PlantUML if true C4 is needed).

## spec/design/ — design is a spec subtype

Design (layout / color / type / motion) can't be expressed precisely in Markdown, so under `spec/design/` the `.json` (tokens) and `.html` (comps) files carry the truth directly — but the two differ: token JSON is the **diffable contract** (the true SOT), an `.html` comp is **reference-grade intent** (not pixel-truth). The structure (flat, graduating to subfolders when it grows), the layer taxonomy, the DTCG token model, motion (global tokens + local choreography), the tool-free design standard, and the content boundary are in **`references/design-system.md`**. Read it before writing any design doc.

## Archive rule (spec vs decision — they differ)

Follows the lifetime class (global rule in `../ctx/references/consistency.md § Archive mechanics`):

- **spec (LIVING, detailed):** localized change → **edit in place**; **foundational** rewrite → move the WHOLE old spec to `spec/archive/` under the canonical archive filename (defined once in `../ctx/references/consistency.md § Archive mechanics`) and write ONE clean new `<name>.md` reusing the canonical name (the active spec stays a single elegant current-truth doc — never a pile of deprecated sections). Record the *why we rewrote* as an ADR. Ripple-fix references in the same commit.
- **decisions (APPEND-ONLY):** **never archived** — supersede in place (above). The trail is the value.
- Archive folder is per-folder, plain name `archive/`, verbatim move + pointer, never delete by default.

## Hard constraints (spec authoring)

> The MUST/NEVER lines below are the **domain** constraints for spec / ADR / design authoring — binding, load-before-act; they fire when a spec, decision, or design doc is written. The **cross-cutting** rules (single-source, same-change, verify-canonical, the gate) live in [`../ctx/references/consistency.md`](../ctx/references/consistency.md) and apply on top. The formats/templates above teach; these lines bind.

### Status & why-discipline

- **MUST** write a spec as **conclusions + constraints** (normative content). **NEVER** embed teaching, tutorials, discursive narrative, or option-comparison in a spec — that is Explanation bleeding into a normative doc; move it to a `reports/` doc (for review) or an ADR (the why).
- **MAY** carry **INTENT** inline — a one-clause *so-that* that scopes a rule so the agent generalizes correctly (`X, so that Y`); this is the one "why" that stays, because deleting it changes edge-case conformance. **MUST NOT** embed **RATIONALE** ("why we chose X over Y") → that belongs in the ADR's `Considered options` + `Decision outcome`. **MUST** cut **BACKGROUND** (context the model already knows / history that changes no action).
- **NEVER** treat a figure, example, or note as a binding requirement — illustrative content is informative, not normative. State the normative claim as a MUST/SHALL line; do not let a diagram imply it.

### Testable, versioned, machine-readable

- **MUST** use **testable acceptance criteria** (EARS `WHEN <trigger> the system SHALL <response>` / Given-When-Then) with **quantified thresholds** — one clause = one test, binary pass/fail. **NEVER** an unfalsifiable criterion ("should be fast" → "P95 < 200 ms").
- **MUST** give **versioned tech choices** (`Postgres 16`, not "a database") and **machine-readable contracts** (interfaces / signatures / endpoints / event schemas / data models with types).
- **MUST** specify the constraint, **not** the mechanism — no hand-written low-level design ("session lookup must be O(1)", not "use a HashMap"). State non-goals / out-of-scope explicitly (an agent can't infer them from omission).

### Traceability & drift recovery

- **MUST** keep the ID chain traceable end-to-end — story → requirement (`REQ-###`) → design (spec) → task (`T###`, in `progress/`), immutable, never reused, followable. **NEVER** ship a requirement whose acceptance criterion no task will implement, nor (in `progress/`) a `T###` task that traces to no `REQ-###`.
- **MUST**, on agent misbehavior, fix the **spec first** then re-run — the spec is the artifact, code is its current expression; **NEVER** patch code past a stale spec or treat the spec as a one-shot prompt.

### No dangling pointers

- **MUST NOT** leave a **dangling pointer** — e.g. "see Q1–Q5" / "per §4" / "→ ADR-0007" where the target is undefined, unwritten, or unresolved. Every forward reference in a spec resolves to a defined, current target in the same commit. (Ties to consistency.md Rule 1's stale-pointer gate.)

### ADR discipline

- **MUST**, before an architecturally-significant choice, **scan `decisions/` + the reject-log** — do not re-litigate a settled decision or reintroduce a previously-rejected concept. (The read-side payoff of the append-only trail; global habit in `../ctx/SKILL.md § the read-side habit`.)
- **MUST** have an ADR record **only DECIDED choices** — a made choice + its real alternatives + consequences. **NEVER** park undecided options as a "proposed" ADR to be filled in later; undecided deliberations stay in `progress/` (or a spec's `[NEEDS CLARIFICATION]` block) until decided, then land as an `accepted` ADR.
- **MUST** number ADRs `NNNN` (4-digit, sequential, monotonic, **never reused**); one decision per file; reverse only by supersede-in-place (bidirectional status flip, old body untouched — mechanics above).

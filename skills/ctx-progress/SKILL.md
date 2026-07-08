---
name: ctx-progress
description: Tracks the WORK truth of a project — where we are, what's next, and what to read first — in a living progress file a fresh session can resume from, including cross-session handoff. Use when updating work or project state in a ctx knowledge base, recording what's done or what's next, opening a progress subtree as it grows past a flat file, handing off to a new session, or auditing a project for progress drift.
license: MIT
metadata:
  author: motiful
  version: "1.0"
---

# ctx-progress — the LIVING work-truth doc

> The lifetime model and shared conventions live in [`../ctx`](../ctx/SKILL.md); this is the how-to for the LIVING `progress/` folder.

## Execution Procedure

`progress/` is the **supervisor's clipboard**, not a second spec: it holds the WORK truth (where we are / next / read-first) and **POINTS at `spec/` + `decisions/`, never restating a product fact**. The durable "what/why" lives in spec/decisions; progress tracks *doing it against them*. **The ordered `T###` task list lives here** (not in the spec — ctx-spec §14 was retired); each `T###` points back to the `REQ-###` in `spec/` that it implements.

```
track_work_truth(project) → one LIVING current-position a fresh session resumes from

# EP1 — Locate / create the progress root
root = ctx/progress/                      # fixed, exactly ONE (see Hard constraints)
doc  = flat progress.md UNTIL ~150 lines, then graduate to a tree

# EP2 — Update work state (the common case)
on any decided work-state change (active leaf moved · next settled · user lock · subagent report):
    write_into_progress(change)           # sink SAME-TURN; never leave decided state only in chat
    reference(spec, decisions, by="pointer")   # never restate a product fact
    # the ordered T### task list + each task's status live HERE (not in spec); every T### → a spec REQ-###

# EP3 — Open a subtree (a node passes ~150 lines)
open_child(); parent = one-line summary + link; never duplicate child content into parent

# EP4 — Pre-reset checkpoint (before /compact, a context reset, or dispatching an independent session)
sink_decided_state()                      # anything decided this session → progress/ (+ spec/decisions if product truth)
verify(active_leaf_current ∧ next_step_written ∧ nothing_decided_only_in_chat)   # progress IS the handoff; no separate prompt artifact

# EP5 — Incoming session (resume)
read(CLAUDE.md → progress root → active node → its linked spec/decisions); summarize; WAIT for confirmation before work

# EP6 — Audit for drift
assert one_active_leaf ∧ frontmatter_valid ∧ pointers_not_restated ∧ thresholds_respected → fix before handoff

# Before committing — apply the cross-cutting constraints
apply("../ctx/references/consistency.md")   # single-source · same-change · verify-canonical · gate + conventions

# DoD per work chunk: commit → update progress → archive-when-long  (see Hard constraints)
```

## Where it lives (no location matrix)

In this collection the location is **fixed**: the knowledge root's `ctx/progress/`. For a shippable artifact whose docs must not ship, it is `<name>-ctx/progress/` (that external `<name>-ctx/` folder IS the knowledge root — no nested `ctx/`). There is no per-archetype location matrix to consult — ctx already fixed `ctx/` as the root, so `progress/` is unambiguous.

`progress/` is a LIVING doc: current focus + next + "read first". It is the WORK truth; `spec/` + `decisions/` are the PRODUCT truth. Two living docs, different domains, no SSOT clash — **guarded by the pointer rule below**.

## Single file, or a tree (JBGE)

Start with **one file** `ctx/progress/progress.md`. Graduate to a **tree** only when the flat file would exceed **~150 lines** — never pre-open a tree. (JBGE: a single file is enough until it demonstrably is not; a premature tree is empty ceremony.)

**There are exactly TWO size thresholds** (this is their single canonical statement — companions reference these numbers, they do not restate divergent ones):

| Threshold | Trigger | Action |
|---|---|---|
| **~150 lines** | a flat `progress.md` grows past it | **graduate flat → tree** = open the first child. ("graduate to a tree" and "open a child" are the SAME event, not two thresholds — this is the point that removes the three-numbers illusion.) |
| **~200 lines** | any single node (flat file OR a tree node) grows past it | **archive** the completed portion per the LIVING archive mechanic (see DoD below). |

- **Flat**: `ctx/progress/progress.md` (frontmatter + current position + next + pointers).
- **Tree**: `ctx/progress/README.md` (entry) + `NN-<slug>.md` children (zero-padded, creation-ordered per consistency.md numbering). A child that itself grows gets a sibling `NN-<slug>/` dir with its own `README.md`. Recommended nesting ceiling: **2 levels** below the root; a 3rd level usually means the work should be factored out as its own project.

**Frontmatter on every progress doc** (this is the canonical field set):

```yaml
---
title: <human-readable, one line, no markdown>
created: YYYY-MM-DD        # absolute ISO date, never relative ("yesterday")
updated: YYYY-MM-DD        # bump on every substantive edit
status: draft | active | stale | archived
---
```

Optional when useful: `parent: ../README.md`, `owner`, `supersedes`. Status is one-way in practice: `draft → active → stale → archived`.

## Exactly one active leaf (per track)

**[LOAD-BEARING — hard constraint, see § Hard constraints below]** Within a **single track** (one workflow / session / subtree), a well-formed progress tree has **exactly one `active` leaf** — the current position; more than one *in the same track* means status propagation failed, so repair it. This is what lets a fresh session find "where we are" in one hop.

When several tracks run in **parallel** — independent sessions on the same repo, the common multi-session case — the invariant is **per-track, not global**: a project has **as many active leaves as it has live tracks**, one per subtree. Parallel work graduates the flat file to a tree so each track owns its **own subtree file**, so that sessions touch different files, never one shared body — that is what keeps a single SOT collision-free when many sessions write at once. The **root index** lists the active tracks and is updated by **append / status-flip, never rewritten by N sessions at once**. A fresh session resumes a track from plain language ("continue X") — the agent opens or resumes that track's subtree; no handshake or lane declaration is needed. *(Running the code, isolating it, and the dev server are the ops layer's job, not progress tracking's; progress only records where a track is and — when useful — where its shared runtime is, to attach.)*

## Parent → child by POINTER, never copy

The parent carries a one-line summary + a link into the child subtree; the detail lives only in the child. **Never duplicate child content into the parent** — copy-style references always drift; pointer-style cannot. The parent surfaces the child's *status* (that's the one piece of child-derived state an index must carry), not its body.

```markdown
## 02 · Stripe integration — status: active

See [`./02-stripe-integration/`](./02-stripe-integration/README.md) for the subtree.
Current position: webhook signature validation (2/4 e2e tests passing). Next: Stripe Tax for EU.
```

## Pointer, don't restate (the SSOT guard)

**[LOAD-BEARING — hard constraint, see § Hard constraints below]** Progress **references** spec and decisions by pointer; it **never restates a product fact**. "Where we are on the auth flow" belongs in progress; "the auth flow requires PKCE" belongs in `spec/`, and progress links to it. Restating the spec fact in progress creates a second copy that goes stale — the exact rot ctx exists to prevent.

## Session continuity — progress IS the handoff

There is **no separate handoff artifact**. A well-maintained `progress/` already carries everything a fresh session needs, so continuity is just two moments — a pre-reset checkpoint out, and a guided read in.

**Pre-reset checkpoint (outgoing)** — before a `/compact`, a context reset, or dispatching an independent session, run one gate: **sink any state decided this session** into `progress/` (and settled product truth into `spec/`/`decisions/`), then **verify progress reflects the current position** — active leaf current, next step written, nothing decided sitting only in chat. That is all "handoff" is now: because state hits disk continuously (sink same-turn below), progress is always current, so a reset is safe at any moment. Do **not** generate a standalone handoff-prompt file — if a fact isn't in progress, fix progress, don't paper over it in a prompt. *(The compaction mechanism itself — how a tool summarizes history — is the tool's concern, not this skill's; ctx owns only that the state is externalized. Optional Claude-Code convenience: a `/ctx-compact` command can trigger this checkpoint before you run `/compact` — the checkpoint is tool-agnostic, the command is just a shortcut.)*

**Incoming (resume)** — read in this order, then stop and confirm:

```
1. CLAUDE.md (+ CLAUDE.local.md if present)   → project instructions
2. ctx/progress/README.md (or progress.md)    → the current active node
3. the active-node file                        → the concrete next step
4. files the active node links to (cap ~3)     → upstream spec/decision context
→ summarize goal / current position / next step to the user, WAIT for confirmation. Do NOT start work first.
```

Do not read the whole tree — it preserves history verbatim on purpose; reading all of it burns the context the session needs to work.

## Sink same-turn (decided state hits disk before the turn ends)

Work-state that got **decided this turn** — the active leaf moved, a next step settled, a user verbal lock ("OK do X" / "lock X"), a subagent's status report — must be written into `progress/` (and any settled *product* choice into `spec/`/`decisions/`) **in the same assistant turn it happens**. Never end a turn, dispatch a subagent, or approach compaction with decided state sitting only in chat. The failure mode this prevents: *chat remembers, files don't; compaction comes, the decision is gone.* A quick end-of-turn self-check — "did the active position, a next step, or a user lock change this turn? then the file changed too" — is cheaper than reconstructing lost state next session.

## Definition of Done (per unit of work)

1. **Commit** the code/doc change.
2. **Update `progress/`** in the same commit: bump `updated`, move the active leaf, ripple the parent's status/pointer line.
3. **Archive when long** — when a progress file/subtree is complete or a node reaches the **~200-line archive threshold** (distinct from the ~150-line graduate-flat→tree threshold), retire it per the **LIVING lifetime-class mechanic** in **[`../ctx/references/consistency.md` § Archive mechanics](../ctx/references/consistency.md)**: localized staleness → edit in place (git keeps the old text); a *foundational* shift → move the whole old doc/subtree to `progress/archive/` under the canonical archive filename and write one clean new doc reusing the canonical name. Do not restate that mechanic or the filename format here — both are defined once there.

## Worked example — a minimal `progress.md`

```markdown
---
title: Acme Auth — Progress
created: 2026-06-20
updated: 2026-07-03
status: active
---

# Auth — where we are

Current position: PKCE token exchange wired; refresh-token rotation is the open piece.
Next: implement rotation + its EARS acceptance test.

Spec: [`../spec/auth.md`](../spec/auth.md) · Why PKCE: [`../decisions/0007-pkce.md`](../decisions/0007-pkce.md)
```

That is the whole shape at flat scale: frontmatter · current position · next · pointers to the SOT (no restated product facts). Grow it into a tree only when it crosses ~150 lines.

## The three lifetime classes

Not restated here — see **[`../ctx/SKILL.md` § The model](../ctx/SKILL.md)**. `progress/` is **LIVING**: edited in place, always current, never carrying a stale section inline.

## Stop condition (JBGE)

The smallest progress surface that lets a fresh session continue correctly. One flat file is the default; a tree is earned, not scaffolded ahead of need. If handoff needs 10+ files or private chat history to continue, the progress doc has failed — reshape it, don't pad the prompt.

## Hard constraints (progress hygiene)

> The MUST/NEVER lines below are the **domain** constraints for progress — binding, load-before-act; they fire when a `progress/` file is touched. The **cross-cutting** rules (single-source, same-change, verify-canonical, the gate) live in [`../ctx/references/consistency.md`](../ctx/references/consistency.md) and apply on top. Applies to every file in a `progress/` tree, a flat `progress.md`, or a `<project>-ctx/progress/`; NOT to skill `references/`, `scripts/`, design docs, or code.

### Location & single root

- **MUST** maintain exactly **one** progress root per project (the knowledge root's `progress/`). Two roots cause drift.
- **NEVER** place progress files under any `.claude/`, `.agents/`, or agent-workspace directory — those are for config, not project state.
- **NEVER** maintain two progress files in one project with mutual cross-references. When a project legitimately spans repos (a shippable artifact + its `<name>-ctx` store), **MUST** declare "Source of Truth" at the top of the authoritative root; other repos edit the SoT only.

### Frontmatter & the pointer rule

- **MUST** give every progress / research / tree-entry doc YAML frontmatter with `title`, `created`, `updated`, `status`; `status` ∈ {`draft`,`active`,`stale`,`archived`} (no synonyms); update `updated` on every substantive edit.
- **MUST** use absolute ISO dates (`YYYY-MM-DD`). **NEVER** relative phrasing (`yesterday`, `Thursday`), **NEVER** filesystem `mtime` as a substitute (clobbered by clone / auto-save / indexers).
- **MUST** have progress **reference** spec/decisions **by pointer** (summary line + link). **NEVER restate a product fact in progress** — a restated fact drifts from `spec/`. (This is the LOAD-BEARING SSOT guard above.)

### Tree graduation & archive DoD

> The two size thresholds — **~150 lines** (flat → open a child) and **~200 lines** (archive a completed node) — are defined once in **§ Single file, or a tree** above. These MUST lines enforce those two numbers; they do not introduce a third.

- **MUST** open a child subtree before a progress file exceeds **~150 lines** (or when a subtask is complex enough to spawn its own session). Reference children by **pointer**; **NEVER** duplicate child content into the parent. Use relative paths in parent/child/archive links. Update the parent summary line in the same commit a child's `status` transitions. Nesting ≤ 2 levels below the root.
- **NEVER** apply an `NN-` prefix to skill `references/`/`scripts/`/`design/` (functional modules, not timelines). **NEVER** rename files to reflect priority — use `status` + the parent summary line.
- **Definition of Done for a work chunk = commit → update progress → archive-when-long.** **MUST** archive completed phases when a file exceeds **~200 lines**, preserving archived text **verbatim** (move, not delete; canonical archive filename per `../ctx/references/consistency.md § Archive mechanics`). Archive a section only when all its sub-tasks are complete. **NEVER** summarize/paraphrase/rewrite when archiving; **NEVER** delete completed history.

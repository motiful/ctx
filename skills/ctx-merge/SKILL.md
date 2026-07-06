---
name: ctx-merge
description: Converges many scattered sources — dated research notes, subagent outputs, audit reports, revision cycles — into one living source of truth without silently dropping or distorting anything, routing each conclusion to exactly one home via a visible disposition ledger and surfacing conflicts as choices for a human. Use when merging or consolidating notes/reports into a ctx source of truth, integrating subagent research, synthesizing multiple audit reports, or closing a decision cycle where alternatives existed. Not for writing a single fresh doc from scratch — use ctx-spec.
license: MIT
metadata:
  author: motiful
  version: "1.2"
---

# ctx-merge — Converge Without Losing or Distorting

> Routing destinations (spec / decisions / scratch) follow the lifetime model in [`../ctx`](../ctx/SKILL.md).

## Execution Procedure

```
converge(sources) → living_kb_update + conflict_choices

# STEP 1 — Extract (provenance starts here)
claims = []
for src in sources:
    claims += extract_atomic(src)          # each conclusion decontextualized, tagged {source, span}

# STEP 2 — Cluster + relate (map-reduce, NEVER a recursive prose merge)
clusters = cluster_paraphrastic(claims)    # dedupe equivalents
for c in clusters:
    relate(c)   # entail → keep one + merge provenance
                # neutral/complementary → keep both
                # contradiction → DO NOT reconcile → promote to a decision point

# STEP 3 — Ledger (make every disposition explicit; a drop is a recorded decision)
ledger = disposition_ledger(claims)        # each: keep→spec/§ | keep→decisions/NNNN | superseded | drop+reason
assert every_source_claim_has_a_disposition(ledger)   # GATE — no invisible absences

# STEP 4 — Assemble (information SUPERSET, only-more-never-less) + conflict register
draft = union(non_conflicting) + conflict_register     # every line carries its source

# STEP 5 — Human adjudication (LLM induces, human judges)
choices = choice_cards(conflict_register)  # choose A / B / keep both / UNSURE + comment
apply(human_clicks(choices))               # non-conflicting superset defaults to keep

# STEP 6 — Faithfulness audit (MANUAL discipline — there is no faithfulness_audit() tool)
#   YOU MUST re-decompose the draft and check every claim back to a source, using a
#   DIFFERENT agent/model than the merger (a model grading itself shares its blind spots).
#   This is a step you perform, not a function you call. See "Faithfulness audit" below.

# STEP 7 — Verify the five constraints, then sink
#   Walk the five-constraint checklist by hand (named target · coverage split · boundary ·
#   destination · reject-checked); fail any → fix the gap, re-walk. Then sink:
sink(draft, ledger)                        # edit spec in place / append decisions per destination
apply("../ctx/references/consistency.md")  # single-source · same-change · verify-canonical · gate — before committing
```

> The lines above are a **procedure you execute by hand**, not an API. `faithfulness_audit` and `five_constraints_hold` name disciplines you must carry out (below) — no such tool exists; do not treat them as callable.

## The two failure modes (why naive merging fails)

- **False negative = silent drop.** A high-value conclusion gets dropped. **Invisible** — you can't see a missing thing by reading the output; it surfaces later at build time as "we keep making the same mistake." Empirically the *harder* class to catch (LLM recall ≪ precision; summarizers drop key items routinely).
- **False positive = stain.** Wrong or trivial content kept as if true. Visible, but only on careful read.

Human merging worked because of unspoken tacit judgment (Polanyi: "we know more than we can tell"). An agent can't replicate that, so you MUST replace it with an explicit harness — not a smarter prompt.

## The merge pipeline (map-reduce, NEVER recursive)

1. **Extract** atomic, decontextualized conclusions from each source, each tagged `{source, span}`. Provenance starts at step 1.
2. **Cluster** equivalent/paraphrastic conclusions (dedupe).
3. **Relate within a cluster:** entailment → keep one + merge provenance; neutral/complementary → keep both; **contradiction → do NOT reconcile — promote to a decision point.**
4. **Assemble** = union of non-conflicting conclusions (information SUPERSET, only-more-never-less) + a **conflict register**, every line carrying its source.
5. **Faithfulness audit (a discipline you perform, not a tool you call):** re-decompose the output and check every claim back to a source — catches silent drops and inventions. **You MUST run this with a *different* model/agent than the merger** (a model auditing its own output shares its blind spots). There is no `faithfulness_audit()` function; it is manual work: dispatch a fresh agent, hand it draft + sources, ask "which source-claims are missing, which draft-claims have no source?", act on what it finds.

**Prompt rule:** instruct the merging agent to emit atomic claims with source IDs and, on conflict, **output both variants tagged CONFLICT — never silently pick one.**

## The disposition ledger (the move that makes drops visible)

Before/while merging, build a ledger: **every source conclusion gets an explicit disposition**, so a drop is a *recorded decision*, not an invisible absence.

| Conclusion (+ source) | Disposition | Destination |
|---|---|---|
| … | keep | → `spec/X.md §…` |
| … | keep | → `decisions/NNNN` |
| … | superseded by … | (chain note) |
| … | **drop** + one-line reason | — |

Routing destinations, per the SOT model (`../ctx/SKILL.md § the model`): **keep → `spec/` (current truth) or `decisions/` (a choice + why)** — the SOT is generative, so what's kept lands in the generative core; **drop → the reject log** (with a one-line reason, so it can't quietly return under a new name); **unsure → an open question** carried to the next round. Raw source material itself is not a merge destination — it already lives in `scratch/` (the model's single home for ALL raw: notes, prompts, research dumps, comparisons). No `log` destination — "what happened" is git. The ledger is the artifact a human reviews; review **judgments**, not absences.

**The reject log lives under `decisions/`** (this is its single canonical home — every other skill references it, none relocates it). A rejection is decision knowledge — "we considered X and rejected it because Y" — so it belongs in the APPEND-ONLY class, not `scratch/`: either an ADR whose status is `rejected`, or a single `decisions/rejected.md` ledger. Keeping it beside the decisions is what lets the constraint-5 reject-check (below) find it, and stops a rejected concept quietly returning under a new label.

**Memory-defense routing.** Every durable conclusion this merge produces routes to a **git-tracked ctx SOT file** (`spec/` / `decisions/`) — **never** to an agent's volatile memory (`~/.claude/projects/*/memory/`, `MEMORY.md`, and equivalents). Memory is not shareable, not in git, not in any dependency chain: a conclusion sunk there dies at the next machine/instance switch — a silent drop by another route. Memory's only legitimate use is a local *pointer* for context recovery ("read `spec/X.md` to restore state"), never the home of the knowledge itself. If a system prompt nudges "save this to memory," sink it to the SOT and leave a pointer instead.

## Human adjudication (LLM induces, human judges)

The LLM does induction/organization (cheap); the human supplies the value judgment (Agrawal/Gans/Goldfarb). So: agent merges into a superset + surfaces conflicts as **pre-structured choice cards** (choose A / choose B / keep both / unsure); human clicks. Non-conflicting superset content defaults to keep.

**Adjudication gate (confidence × blast-radius).** Decide *which* dispositions you apply yourself vs. escalate by one rule: **high-confidence + low-blast dispositions auto-apply; anything high-blast OR low-confidence escalates to the human as a choice card.** A non-conflicting paraphrase-dedupe you're sure of = auto-apply. A contradiction, a supersede that flips a LOCKED decision, or a drop you're unsure about = escalate. (This is the merge-side instance of the same gate ctx-report uses for its verdict.)

- **Allow "unsure" + free comment as first-class** — never force yes/no. People discover their criteria *while* grading (criteria drift, Shankar UIST 2024); a rigid binary forces wrong buckets.
- `unsure` + commented items resurface as the next round's decision points. The ledger is replayable: a re-merge honors prior keep/drop/superseded.
- A lightweight single-file HTML reader (renders the merged doc, one decision-card per block/conflict, writes back a `decisions.json` ledger) is the intended tool. Serve over http, not `file://`.

## The five constraints (every merge/deferral MUST satisfy all)

The merge artifact is read by future agents without your context. Implicit answers are forbidden.

1. **Named target** — name the exact doc/section content is merged into. Never "the spec."
2. **Coverage split** — for each source, state what is covered vs not; flag uncovered as "dropped (reason)" or "still open (tracked where)." A partial merge with no gap-flag is indistinguishable from a complete one — the exact failure to prevent.
3. **Boundary disambiguation** — when applicability differs by case, name the trigger variable. No "decide case-by-case later."
4. **Implementation/destination detail** — cite the destination (file/section) and change kind (insert/replace/split). No bare "update."
5. **Reject-concept cross-check** — cross-check every merged-in proposal against the project's reject log before adopting. The subagent doesn't know the rejection history; you do. Watch rename traps (same mechanism, new label). Subagent output is a candidate, not source-of-truth.

## Deferral discipline

Deferring = merging into the future-backlog. Same five constraints, plus: name the backlog file AND write the entry in the same operation (no phantom deferral); record the **un-defer trigger** ("after X exists"); never split "half deferred, half locked" in one merge.

## Verification (before declaring a merge done)

Walk this checklist **by hand** — it is a discipline, not a test suite. Nothing here is a callable assertion; you read each line and confirm it holds.

- For each merged conclusion: named-target ∧ coverage-split ∧ boundary ∧ destination ∧ reject-checked.
- For each deferral: target-file-written ∧ un-defer-trigger recorded.
- Reject log named ∧ no silent partial merges ∧ faithfulness audit run by a *different* agent and its findings acted on.

Fail any → not done. Fix the gap, re-walk.

## Honest limit

Drops can be reduced, not eliminated (omission is empirically the hardest to detect; verifiers share the generator's blind spots; provenance lowers but doesn't zero false positives). The ledger's job is to convert invisible absences into recorded decisions — the human still spot-reads originals at high-stakes points.

---
name: ctx-merge
description: Converges many scattered sources — dated research notes, subagent outputs, audit reports, revision cycles — into one living source of truth without silently dropping or distorting anything, routing each conclusion to exactly one home via a visible disposition ledger and surfacing conflicts as choices for a human. Use when merging or consolidating notes/reports into a ctx source of truth, integrating subagent research, synthesizing multiple audit reports, closing a decision cycle where alternatives existed, or rolling a corpus too large for one context through successive batches. Not for writing a single fresh doc from scratch — use ctx-spec.
license: MIT
metadata:
  author: motiful
  version: "1.4"
---

# ctx-merge — Converge Without Losing or Distorting

> Routing destinations (spec / decisions / scratch) follow the lifetime model in [`../ctx`](../ctx/SKILL.md).

## Execution Procedure

```
converge(sources) → living_kb_update + conflict_choices

# STEP 0 — Scale check (before anything else)
if sources exceed one context, or constraint 2 (coverage split) is unexecutable at this scale:
    run this ENTIRE procedure once per batch    # see § Rolling batches
    # batches share ONE carried-forward framing; each batch still runs STEP 1–7 in full

# STEP 1 — Extract (provenance starts here)
claims = []
for src in sources:
    claims += extract_atomic(src)          # each conclusion decontextualized, tagged {source, span}

# STEP 2 — Cluster + relate (map-reduce, NEVER a recursive prose merge — see § What "NEVER recursive" bans)
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

# STEP 7 — Verify the four constraints, then sink
#   Walk the four-constraint checklist by hand (named target · coverage split · boundary ·
#   destination); fail any → fix the gap, re-walk. Then sink:
sink(draft, ledger)                        # edit spec in place / append decisions per destination
apply("../ctx/references/consistency.md")  # single-source · same-change · verify-canonical · gate — before committing
```

> The lines above are a **procedure you execute by hand**, not an API. The faithfulness audit and the four-constraint walk name disciplines you must carry out (below) — no such tool exists; do not treat them as callable.

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

### What "NEVER recursive" bans — and what it does not

It bans **recursive prose compression**: summarizing a summary, then merging that summary with the next source, so that whatever finally decides never sees the original wording. Every reduce step reads source-tagged atomic claims, never a previous step's prose.

It does **NOT** ban handing every mapper the same shared context — a carried-forward framing, a glossary, the current conclusion set. That is not compression, it is a common vocabulary, and the mappers still read their own sources verbatim. Reading this line as *"each mapper must work blind"* produces piles of mutually untranslatable fragments and pushes the translation onto a reduce step that never read the originals — which is the exact loss this skill exists to prevent, arrived at by way of obeying it.

## Rolling batches (when the corpus exceeds one context)

**Trigger.** The sources do not fit one context, or constraint 2 (coverage split — state per source what is covered vs not) cannot actually be executed at this scale. Against 35 sources that constraint is unenforceable; against 3 it is natural. That is the tell.

**Shape.**

1. **Order the sources newest-first.** The newest carry the current vocabulary, and they become the lens the older ones are read through.
2. **3–4 sources per batch.**
3. **Batch 0 produces the `framing`** — a structured statement of the current conclusions, written in the *destination* format (so the format gets its first real test on the smallest batch, not after the last one).
4. **Every later batch takes `previous framing + its own sources`** and returns an updated framing.
5. **The entire procedure above runs INSIDE each batch.** N batches = N complete extract→cluster→ledger→assemble→adjudicate→audit cycles. This is not a decomposition of the procedure — it is the procedure at the scale it was designed for.

**The comparison object is `claim ↔ framing`, not `claim ↔ claim`.** Each batch's agent holds the already-converged half and reads the old sources through it, translating yesterday's wording into today's on the spot. There is no "organize it all at the end" step, because organizing started at batch 0.

**The framing is a SUPERSET per batch** (STEP 4): a batch only adds. Removing something requires a conflict card — never an in-passing edit.

**The structural defect, and its hedge.** The framing evolved out of these same sources, so it inherits their blind spots; filtering old material through current conclusions is **structured confirmation bias**. That is the price of rolling, and it is real. The hedge is the mandatory `unplaceable` field (below), which asks the opposite question — *what is in here that the framing has no place for?*

**Audit every batch, not once at the end.** An end-of-run audit has to re-read the whole corpus to answer "which source claim never arrived" — the same arithmetic that forced batching in the first place, merely postponed. Per batch, the audit reads only this batch's sources plus the framing delta, and STEP 6's "must be a different agent" is satisfied for free by opening a fresh one each batch. It asks exactly two questions, which are the two ways a rolling merge loses things:

- **Q1 — which conclusion in this batch's sources never reached the framing *and* has no disposition in the ledger?** The classic silent drop.
- **Q1b — for each claim the ledger dispositions as `keep → <destination>`, go to that destination and read: is the content actually there?** A ledger row asserting a landing is the one kind of absence an auditor will not go looking for — the row itself says the checking is done. Empirically the sharpest of the three: in the first batch that ran this procedure, four of eight losses were catchable *only* by this question, and every one of them sat behind a row reading `keep`. **Read the destination; do not grep for it.** A merger that self-checked by grepping each destination reported 59/59 landed; an auditor who opened 30 of them found 9 empty. Grep proves the heading exists, which is not the claim being made. It errs in both directions, too: a later batch got a zero for content that *was* there, because the search string came from the ledger's wording rather than the framing's — and acting on that zero would have written the passage in twice.
- **Q1c — for each assertion carrying normative weight in the framing, go find the sentence in the sources: is it there?** Q1 and Q1b both hunt the *silent drop*; this one hunts the other failure mode this skill names — the **stain**. Without it the audit tests one of the two and reports clean. In the first batch it was the only question that caught six fabrications, among them an unsourced design decision and a `MUST` contradicting its own figure two lines above.
- **Q2 — which entry present in the previous framing is absent from this one?** Squeezed out during carry-forward — the failure mode rolling has that a single pass does not. Mechanize it: keep the framing in git, and `git diff`'s deletion lines are the candidate list; the auditor only judges "replaced by better wording" vs "gone".

Any question returning a finding → **the batch is not done.** Backfill, then re-audit.

**Backfill discipline — the second pass has its own two failure modes.** Both were observed on the first batch ever to run this procedure, in this order:

- **The fix injects what the sources never said.** Round one dropped content; round two invented it. Filling a hole is generative work, and a merger that marks its inferences scrupulously in normal operation will state them flatly while patching. So the re-audit MUST ask *"does the backfill contain anything with no source?"* — not merely re-check the original findings — and the backfilling agent MUST be told to mark an inference as an inference. **This is the worse direction: an invented claim leaves no trace that it was ever absent, while a dropped one at least still exists in the source.**

  One batch fabricated once per repair round, weakening each time: **a fact** (a claim about a source, refutable by one grep), then **a reading** (a reconciliation the source never proposed, in the same argumentative slot), then **a count** (a number where the source gave none, headed by the word "verbatim"). None was a knowledge error. All three filled the same kind of gap: a passage of reasoning that needed one more piece to read as finished, where supplying it was easier than leaving a hole. That makes it a property of the writing, not of the writer, so "be careful" does not address it. What does: **three sentence shapes account for all of them — the reconciliation ("these don't actually conflict", "it's a division of labour"), the count ("N locations", "all four"), and the connective ("taken together", "therefore"). Reaching for any of them is the cue to go back to the source, because each one asserts something no single quoted line contains.**
- **The fix patches the instance, not the class.** When a finding names two examples of one pattern — say, the truncated second half of a bulleted source note — the backfill repairs exactly those two and leaves the rest of the pattern standing. Name the *class* in the finding and require a re-sweep of it, or the same shape returns next batch wearing a different line number.

## Source reliability — declare it before merging

Rank the sources into explicit tiers *before* the first batch, and write the ranking down. Merge without a declared ranking and every source argues as an equal: a superseded note from months ago contradicts the latest conclusion, and the merge stops for an adjudication the ranking would have settled.

- A contradiction **between** tiers resolves toward the higher tier automatically, recorded as `superseded` with the tier as its reason. It does not become a conflict card.
- A contradiction **within** a tier is a real conflict and escalates (STEP 5).
- The ranking is a claim about the sources, not about the truth. State it explicitly so a later reader can disagree with it explicitly.

## The disposition ledger (the move that makes drops visible)

Before/while merging, build a ledger: **every source conclusion gets an explicit disposition**, so a drop is a *recorded decision*, not an invisible absence.

| Conclusion (+ source) | Disposition | Destination |
|---|---|---|
| … | keep | → `spec/X.md §…` |
| … | keep | → `decisions/NNNN` |
| … | superseded by … | (chain note) |
| … | **`superseded-by-newer`** — same thread, higher revision wins | — (does **NOT** enter the conflict register) |
| … | **`copied-as-is`** | → destination, unchanged |
| … | **drop** + one-line reason | — |

Routing destinations, per the SOT model (`../ctx/SKILL.md § the model`): **keep → `spec/` (current truth) or `decisions/` (a choice + why)** — the SOT is generative, so what's kept lands in the generative core; **drop → recorded in the ledger** with a one-line reason — the ledger is replayable, so a re-merge honors a prior drop and a rejected concept cannot quietly return under a new label; **unsure → an open question** carried to the next round. Raw source material itself is not a merge destination — it already lives in `scratch/` (the model's single home for ALL raw: notes, prompts, research dumps, comparisons). No `log` destination — "what happened" is git. The ledger is the artifact a human reviews; review **judgments**, not absences.

**Never dispose of a whole section as `superseded`.** Supersession is a claim about a *claim*, not about a region of text. A section usually contains several, and a later revision rarely overturns all of them — most often it revises two and relocates a third. Disposing of the region marks the survivors as handled, and the ledger then reports the section as processed, so no later batch comes back for them. **A source that says of its own content "this did not disappear, it moved" is naming a destination: record the destination, or open the question of where it went. What is forbidden is `superseded` with no receiver** — the one form of the disposition that cannot be checked, because there is nothing to compare against. This is how a batch lost two rules that its own source had explicitly declared surviving.

**`superseded-by-newer`.** Within one revision thread, the higher-numbered revision wins **automatically** and the difference does not become a conflict card. Without this, a 22-revision thread manufactures hundreds of false conflicts and adjudication collapses under its own volume. It holds only *within* a thread — across threads, or between same-tier sources, a contradiction is still a contradiction and STEP 2 applies.

**`copied-as-is`.** Copying is a disposition and gets its own line. It is not "skipped": a skip is invisible, and an invisible skip reads exactly like *"I looked at it and it was fine."*

**One row, one claim.** A row that joins two items — "the term *plus* the four-row comparison table" — will be verified on the first and the second dies in silence, because every check that follows looks for evidence the row landed and finds it. This survived a merger's self-check, a second self-check by a corrected method, and an auditor's first sampling pass; three independent checks all stopped at the conjunction's left half. Split at extraction, not at verification: by the time anyone is checking, the row already reads as one thing.

**Count the ledger by machine, and record the command.** The tally is the only visible evidence that `every_source_claim_has_a_disposition` held, so an asserted number is evidence of nothing. Two consecutive batches reported counts that were wrong — one whose total silently treated six unextracted claims as nonexistent, one that reported three mutually inconsistent numbers (rows, self-report, and the sum of its own categories). A number nobody can reproduce is worse than no number: it reads as verification.

**`unplaceable` — a mandatory ledger field, not a disposition.** Every batch ledger MUST answer: *is there anything in these sources the current framing has no place for?* An empty answer is acceptable **only** as an explicit sentence ("read through; nothing the framing cannot hold"). A silently empty list and a carefully-searched empty list read identically, and only one of them is true. Whatever lands here is the highest-value line in the ledger — it is precisely what the framing's blind spot would otherwise have filtered out.

**Memory-defense routing.** Every durable conclusion this merge produces routes to a **git-tracked ctx SOT file** (`spec/` / `decisions/`) — **never** to an agent's volatile memory (`~/.claude/projects/*/memory/`, `MEMORY.md`, and equivalents). Memory is not shareable, not in git, not in any dependency chain: a conclusion sunk there dies at the next machine/instance switch — a silent drop by another route. Memory's only legitimate use is a local *pointer* for context recovery ("read `spec/X.md` to restore state"), never the home of the knowledge itself. If a system prompt nudges "save this to memory," sink it to the SOT and leave a pointer instead.

## Human adjudication (LLM induces, human judges)

The LLM does induction/organization (cheap); the human supplies the value judgment (Agrawal/Gans/Goldfarb). So: agent merges into a superset + surfaces conflicts as **pre-structured choice cards** (choose A / choose B / keep both / unsure); human clicks. Non-conflicting superset content defaults to keep.

**Adjudication gate (confidence × blast-radius).** Decide *which* dispositions you apply yourself vs. escalate by one rule: **high-confidence + low-blast dispositions auto-apply; anything high-blast OR low-confidence escalates to the human as a choice card.** A non-conflicting paraphrase-dedupe you're sure of = auto-apply. A contradiction, a supersede that flips a LOCKED decision, or a drop you're unsure about = escalate. (This is the merge-side instance of the same gate ctx-report uses for its verdict.)

- **Allow "unsure" + free comment as first-class** — never force yes/no. People discover their criteria *while* grading (criteria drift, Shankar UIST 2024); a rigid binary forces wrong buckets.
- `unsure` + commented items resurface as the next round's decision points. The ledger is replayable: a re-merge honors prior keep/drop/superseded.
- A lightweight single-file HTML reader (renders the merged doc, one decision-card per block/conflict, writes back a `decisions.json` ledger) is the intended tool. Serve over http, not `file://`.

## The four constraints (every merge/deferral MUST satisfy all)

The merge artifact is read by future agents without your context. Implicit answers are forbidden.

1. **Named target** — name the exact doc/section content is merged into. Never "the spec."
2. **Coverage split** — for each source, state what is covered vs not; flag uncovered as "dropped (reason)" or "still open (tracked where)." A partial merge with no gap-flag is indistinguishable from a complete one — the exact failure to prevent.
3. **Boundary disambiguation** — when applicability differs by case, name the trigger variable. No "decide case-by-case later."
4. **Implementation/destination detail** — cite the destination (file/section) and change kind (insert/replace/split). No bare "update."

## Deferral discipline

Deferring = merging into the future-backlog. Same four constraints, plus: name the backlog file AND write the entry in the same operation (no phantom deferral); record the **un-defer trigger** ("after X exists"); never split "half deferred, half locked" in one merge.

## Verification (before declaring a merge done)

Walk this checklist **by hand** — it is a discipline, not a test suite. Nothing here is a callable assertion; you read each line and confirm it holds.

- For each merged conclusion: named-target ∧ coverage-split ∧ boundary ∧ destination.
- For each deferral: target-file-written ∧ un-defer-trigger recorded.
- No silent partial merges ∧ faithfulness audit run by a *different* agent and its findings acted on.
- **Rolling merge, per batch:** the ledger covers every extracted claim ∧ `unplaceable` answered in words (not a silent empty list) ∧ the audit's Q1, Q1b, Q1c and Q2 all came back clean.

Fail any → not done. Fix the gap, re-walk.

## Honest limit

Drops can be reduced, not eliminated (omission is empirically the hardest to detect; verifiers share the generator's blind spots; provenance lowers but doesn't zero false positives). The ledger's job is to convert invisible absences into recorded decisions — the human still spot-reads originals at high-stakes points.

**Rolling adds one of its own:** the framing is written before most sources have been read, so it is a structured prior. `unplaceable` and the per-batch Q2 audit narrow that, they do not close it. If `unplaceable` runs hot for two consecutive batches, the framing's skeleton is wrong — stop and reshape it rather than continuing to push material into it.

**Where a project's long-term reject log lives is currently unassigned** — the previous answer (a `rejected`-status ADR under `decisions/`) is under revision. Inside a merge the ledger carries the record, and that is enough for the merge itself; a project needing a durable rejection trail must name its own home until this is settled.

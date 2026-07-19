---
name: ctx-compact
description: Sweeps everything decided in the current session into its correct source-of-truth home — progress, decisions, spec — and verifies the knowledge base is current and internally consistent before a context reset, so a /compact or a new session loses nothing. Use before running /compact, before ending a session, or whenever the user says they are about to reset or hand off and wants a safety checkpoint. A thin orchestrator over ctx-progress, ctx-merge, ctx-spec, and the consistency gate — it introduces no new rules, it runs the existing ones as a checklist. Not for routine mid-session saves — that is sink-same-turn in ctx-progress.
license: MIT
metadata:
  author: motiful
  version: "1.1"
---

# ctx-compact — the pre-reset checkpoint sweep

> A user-invoked **safety ritual**, not a new mechanism. It runs the sink discipline that already lives in the companions — `ctx-progress` (tracking), `ctx-spec`/`ctx-merge` (SOT), the consistency gate — across every SOT home at once, so nothing decided this session is left only in chat when context resets. Lifetime model in [`../ctx`](../ctx/SKILL.md).

## Execution Procedure

The risk this addresses is asymmetric and catastrophic: a `/compact` or a new session **loses whatever was decided but never written to disk**. Mid-session that is handled continuously by *sink-same-turn* (ctx-progress); this skill is the **explicit, user-triggered second check** right before the reset, and — unlike sink-same-turn — it is **cross-cutting**: it sweeps not just progress but decisions and spec too.

```
checkpoint_before_reset() → nothing decided is left only in chat; the base is current + consistent

# STEP 1 — Sweep the TRACKING gate (progress) — the common case
result = Skill("ctx-progress")            # run its pre-reset checkpoint (EP4): sink decided work-state;
                                          # verify active leaf current, next step written, nothing only-in-chat
assert result.delivered                   # GATE — do not report "safe to /compact" on an unconfirmed sink

# STEP 2 — Sweep the SOT gate (decisions / spec) — what progress does NOT own
for each thing DECIDED this session that has crossed the SOT gate (defined in ctx-progress § Sink same-turn):
    durable product truth  → spec/            (edit in place — via ctx-spec)
    a made choice + why     → decisions/NNNN   (append — via ctx-merge / ctx-spec)
# an agent-synthesized-this-turn design the owner has NOT reviewed does NOT cross — it stays a report proposal
# (the two gates are defined once in ctx-progress; this step RUNS them, it does not redefine them)

# STEP 3 — Verify the base is current + internally consistent
apply("../ctx/references/consistency.md") # single-source · same-change (incl. code↔doc) · index-sync
                                          # (reports/ + decisions/ READMEs) · verify-against-canonical · the gate

# STEP 4 — Confirm safe to reset
report: what was swept · what is still open (and where it is tracked) · indexes in sync → "safe to /compact"
```

## What it is (and is not)

- **It is** the [`ctx-progress`](../ctx-progress/SKILL.md) pre-reset checkpoint **promoted to a user-invocable, cross-SOT entry** — you say "I'm about to compact" (or run `/ctx-compact`) and it sweeps progress **and** decisions **and** spec, then verifies the indexes, in one deliberate pass.
- **It is not** a new sink rule. Every rule it applies is defined elsewhere: the two gates and sink-same-turn in `ctx-progress`, distillation in `ctx-merge`, ADR/spec form in `ctx-spec`, index-sync and the gate in `consistency.md`. This skill **references** them; it never restates them (single-source).
- **It is not** the tool's compaction mechanism. *How* a tool summarizes history when you run `/compact` is vendor-owned and changes; ctx-compact owns only that the durable **state is externalized first**, so the summary is a convenience, not a single point of failure.

## The two gates it respects

Sink the **tracking** gate every time; sink the **SOT** gate only for what has passed review. Both gates and their exact criteria are defined once in **[`ctx-progress` § Sink same-turn](../ctx-progress/SKILL.md)** — ctx-compact enforces them across all SOT homes at reset time, it does not re-author them. That distinction is what stops this sweep from over-committing a half-formed, unreviewed design just because the session is ending.

## Honest limits

ctx-compact cannot rescue a decision that was **never externalized in any form** — if a conclusion lived only in the agent's reasoning and never surfaced (not even as a report proposal), the sweep has nothing to find. It is a **checklist, not magic**: it makes the pre-reset discipline explicit and user-triggerable, but the discipline still has to run. And it verifies *structural* currency (state on disk, indexes synced) — it does not judge whether the decisions themselves were correct.

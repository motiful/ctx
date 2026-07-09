<!--
  GUIDE = the downstream funnel: ENABLE (for people who've decided to use ctx).
  Organized by Diátaxis: Tutorial (learn) / How-to (do a task) / Explanation (understand).
  Reference lives in each skill's SKILL.md; sales lives in ../README.md.
-->

# ctx Guide

You've decided to use ctx. This is how. Three parts, by what you need right now:

- **[Tutorial](#tutorial--your-first-ctx-in-5-minutes)** — learn it by doing, once, start to finish.
- **[How do I…](#how-do-i)** — recipes for specific tasks.
- **[Why it's shaped this way](#why-its-shaped-this-way)** — the reasoning, when you want to trust it.

---

## Tutorial — your first ctx in 5 minutes

**Goal:** bring one existing repo under ctx and record one real decision.

1. **Install.** `npx skills add motiful/ctx --all`
2. **Adopt.** In your repo, tell your agent: **"apply ctx to this repo."**
   The agent (via `ctx-adopt`) inspects the repo, proposes where `/ctx` lives (an in-repo folder by default), and shows you a **migration plan** — a table of the scattered docs it found and where each would go. Nothing moves yet.
3. **Approve.** Say yes. Now you have:
   ```
   your-repo/
   ├─ ctx/
   │  ├─ spec/         # current truth — what you're building
   │  ├─ decisions/    # ADRs — choices + why, append-only
   │  ├─ progress/     # where we are, what's next
   │  ├─ reports/      # disposable docs a human decides from
   │  └─ scratch/      # raw notes, experiments
   └─ … your code, untouched …
   ```
4. **Record a decision.** Tell the agent: *"we chose Postgres over SQLite because we need concurrent writers — record it."* It writes an ADR into `decisions/` with a status and a reason.
5. **Check.** Open `/ctx`. You have a living spec, one decision with its why, and a progress note — the seed of a base that won't rot.

That's the whole loop: **converge → record → derive → discard.** You just did it once.

---

## How do I…

**…record a decision?**
Tell the agent the choice + the why ("we picked X over Y because Z"). It writes an ADR to `decisions/` (append-only). To reverse it later, you don't edit the old one — you append a new ADR that supersedes it. (`ctx-spec`)

**…converge a pile of dated notes / subagent outputs into the truth?**
"Merge these notes into the spec." The agent extracts atomic conclusions, routes each to exactly one home (spec / decision / drop-with-reason) via a visible ledger, and surfaces conflicts as choices instead of silently resolving them. (`ctx-merge`)

**…track progress across sessions?**
Keep one `progress/` that says where we are + what's next + what to read first. When it outgrows a flat file (~150 lines), it graduates to a tree. A new session reads the active node and picks up. (`ctx-progress`)

**…write a report to get a human decision?**
Ask for a report. You get a numbered, disposable HTML doc that lays out options and ends by asking for a verdict. After you comment, its keep-worthy conclusions are distilled into the SOT and the report is archived. (`ctx-report`)

**…keep ctx out of my open-source repo?**
When `ctx-adopt` detects a published repo, it makes `/ctx` a **gitignored symlink** to an external `../<project>-ctx/`. Your code ships clean; the agent still reads `/ctx`; your secrets never leave. (`ctx-adopt`)

**…stop code and docs from drifting apart?**
When a change alters observable behavior, its spec/ADR updates in the *same* change — not "later." Before marking a task done, sweep the SOT for the changed behavior. (This is Rule 2 — same change, no drift — one of the cross-cutting rules every ctx skill applies before committing.)

**…know when the ctx folder is "done" being set up?**
Run the folder Definition-of-Done checklist — it's Rule 4 (the gate) in the cross-cutting rules, alongside the other standing gates: skeleton exists, each folder has an index, SOT is seeded, no orphan docs, boundary + confidentiality decided. It's both a stop-condition and a standard. (`ctx-adopt`, applying the shared `consistency.md`)

**…keep a dev server running across sessions — or share one across parallel sessions?**
Host it detached in tmux: one instance survives a `/compact`, and every parallel session on the repo shares it instead of each starting its own. The topology (what runs, where, how to restart) lives in a committed `services.md`; whether it's up *right now* is detected on demand, never stored in a file that could go stale. (`ctx-serve`)

**…make sure a /compact doesn't lose what I decided?**
Before you reset, run the checkpoint: it sweeps everything decided this session into its home — progress, decisions, spec — verifies the folder indexes are in sync, and confirms nothing is left only in chat. Say "I'm about to compact" or run `/ctx-compact`. (`ctx-compact`)

---

## Why it's shaped this way

The reasoning is recorded as decisions (ADRs) in this repo's own `ctx` — ctx uses itself. The load-bearing few:

**Why organize by lifetime, not by pipeline stage?**
Stage-based folders (requirements → research → design → decisions) leave a persistent, editable doc at *every* stage — and every one of them goes stale independently, so you get N drifting copies of the truth. Lifetime tells you the one thing you need to act: *can I edit this in place (living), must I never rewrite it (append-only), or can I throw it away (disposable)?* → ADR `0001-organize-by-lifetime-not-stage`.

**Why is the source of truth "generative"?**
If you store explanations, tutorials, and option-comparisons as long-term docs, they become a second pile that rots. So ctx keeps only `spec/` + `decisions/` as truth and *derives* every explanation on demand, then discards it. The base stays lean by construction. → ADR `0002-sot-is-generative-no-explanation-class`.

**Why "convergence, not a wiki"?**
A wiki is *divergent* — independent articles that grow sideways. ctx is *convergent* — messy multi-session exploration pulled into ONE truth you build from. Different goal, opposite shape. → ADR `0007-convergence-not-a-wiki`.

**Why does `/ctx` stay at the repo root even when its bytes live elsewhere?**
Because the agent runs *in the repo* and must always find the context at a predictable place. So the **mount point** (`/ctx`) is fixed; the **backend** (in-repo folder, or a symlink to an external store) is swappable. Stable mount, swappable backend.

**Why a gate before every non-trivial task?**
"Looks done" is not done. ctx writes an acceptance checklist *before* executing and validates against it *after* — the same pattern behind spec acceptance-criteria, progress completion, and report self-verification.

Full decision history lives in this collection's own backstage store — `ctx-ctx/` in the source repository (ctx dogfooding itself). It is not shipped with the installed skills.

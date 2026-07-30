# Contributing to ctx

Thanks for being here. ctx is young and shaped by real usage — if a skill misfires, a doc confuses, or you've found a sharper way to say something, we want it. This guide sets expectations so your effort lands.

> **One thing up front: ctx is an _opinionated_ methodology, not a neutral toolkit.** Most of what makes it useful is what it deliberately _refuses_ to do. Before proposing a change, please read **[Design philosophy — the non-negotiables](#design-philosophy--the-non-negotiables)** below. A PR that fights a core principle will be declined no matter how clean the code — not because it's bad work, but because the constraint _is_ the product.

## What we welcome directly

Open a PR straight away for:

- **Doc fixes** — typos, broken links, unclear wording in the README, the [Guide](docs/GUIDE.md), or any `SKILL.md`.
- **Skill refinements** — tightening a phrasing, fixing a wrong example, closing a small gap in an existing skill's behavior.
- **Bug reports** — [open an issue](https://github.com/motiful/ctx/issues) with what you expected vs. what happened. "This didn't do what I expected" is a valid, useful report.

## What needs an issue first

Please **[open an issue](https://github.com/motiful/ctx/issues) to align on direction before writing code** for anything that changes what ctx _does_ or how it's shaped:

- a new skill, or a new capability in an existing one;
- anything that adds configuration, runtime, state, or a new file ctx has to maintain;
- a change to where/how the `/ctx` backend lives, how it's announced, or how docs are classified;
- anything you'd describe as a "feature."

This isn't bureaucracy — it's so you don't spend an evening on a PR we have to decline on principle. We may propose a different shape, fold the idea into an existing skill, or (if it's genuinely out of scope) decline it with reasons. If it fits, we'll gladly take the PR and credit you.

## Design philosophy — the non-negotiables

These are the constraints a change must respect. They're not preferences; each one is load-bearing, and most have a decision record behind them (see [Where the "why" lives](#where-the-why-lives)).

1. **Stable mount, swappable backend.** `/ctx` is a fixed mount point at the repo root — the agent always reads `<repo>/ctx`. _Where the bytes actually live_ (in-repo folder, external symlink, central store) is a **deployment detail** that stays behind the symlink. Don't surface it into always-loaded skill descriptions or default logic.
2. **Content-only, stateless.** ctx is Markdown and a symlink — no runtime, no config files, no machine-level state it has to maintain. The `/ctx` folder plus your agent's own `AGENTS.md`/`CLAUDE.md` pointer are the _whole_ contract. If a change needs to persist state, that's a signal it belongs in the agent's layer, not in ctx.
3. **Lightest backend by default; upgrade on demand.** The default is a plain in-repo folder — zero setup. ctx externalizes only when there's a real reason (the context must not ship) and only when asked. Never front-load setup or ask a first-run question a new user has no basis to answer.
4. **Organize by lifetime, not pipeline stage.** Every doc is _living_ (edit in place), _append-only_ (never rewrite — supersede), or _disposable_ (throw away). No stage-based folders that each rot independently.
5. **The source of truth is generative.** Truth is `spec/` + `decisions/` only; explanations, tutorials, and comparisons are _derived on demand and discarded_, never stored as a second pile that rots. Don't add a long-lived "explanation" class.
6. **Tool-agnostic.** ctx never forces a vendor. The `/ctx` folder is the neutral contract (like `.git`); the pointer file follows whatever the project/agent already uses.
7. **Decide, then build — and supersede in place.** A design decision is recorded as an ADR _before_ it's built. We never rewrite a decided ADR; we append a new one that supersedes it, so the reasoning is never lost. This is why methodology changes start as an issue: the decision comes first.

If your idea requires bending one of these, the issue thread is the place to make the case — sometimes a principle _should_ evolve, but that's a decision to take deliberately, in the open, as a new ADR.

## Making a change

- Keep PRs **small and single-purpose** — one fix or one refinement per PR.
- **Match the house voice** in docs and `SKILL.md` files: plain, opinionated, present-tense.
- **No drift.** If your change alters observable behavior, update the relevant `SKILL.md` / doc in the _same_ PR — not "later."
- Describe _what_ changed and _why_ in the PR body.

## Where the "why" lives

ctx uses itself: its design decisions are recorded as ADRs. The load-bearing few are explained in **[the Guide → "Why it's shaped this way"](docs/GUIDE.md#why-its-shaped-this-way)**. <!-- TODO(owner V2.2): once the methodology ADRs are published, link docs/decisions/ here as the full rationale. --> If you're proposing a methodology change, read those first so your issue engages with the actual reasoning.

## Getting help

Questions, ideas, "is this in scope?" — [open an issue](https://github.com/motiful/ctx/issues) or start a discussion. We'd rather talk early than have you guess.

<!-- TODO(owner): no CODE_OF_CONDUCT.md exists yet — recommend adding Contributor Covenant and linking it here (GitHub-standard for a public project). -->

---

MIT-licensed — see [LICENSE](LICENSE). By contributing, you agree your contributions are licensed under the same terms.

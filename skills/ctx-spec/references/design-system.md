---
name: design-system
description: Structures spec/design/ — the design subtype of a spec, where .html and .json files carry the truth Markdown can't (layout, color, type, motion). Covers the layer taxonomy (foundations · components · patterns · pages), the DTCG token model, motion as global tokens + local choreography, the tool-free design standard, reference-grade fidelity, the content boundary (what a ctx store holds vs points at), and the per-artifact archive rule. Read from ctx-spec before writing any design doc.
---

# spec/design/ — Design as a Spec Subtype

Design lives close to the user's eyes: layout, color, type, spacing, motion. **Markdown can't express it precisely** — so under `spec/design/` the `.json` (tokens) and `.html` (comps, component prototypes) files carry the truth directly. But the two are not the same *kind* of truth — see *What is actually the SOT* below.

## The layer taxonomy

Design systems across the mature field (Material 3, Carbon, Polaris, Atlassian, SLDS) share four layers:

- **Foundations** — tokens + primitives: color, type, spacing, grid, elevation, motion, iconography, a11y. GLOBAL, one set per project.
- **Components** — reusable UI building blocks (button, card, modal). GLOBAL, one canonical definition each.
- **Patterns** — reusable multi-component solutions to recurring problems (forms, empty states, notifications). A **composition tier**, not a heavyweight peer: a multi-step user flow is a pattern.
- **Pages** — full-screen compositions built from components. LOCAL, per-feature; nobody imports a page.

These are *layers of meaning*, not a mandatory folder tree — the folder shape stays flat until it grows (below).

## Folder structure — flat first, graduate when it grows

Start FLAT; the four layers are concepts, not required folders. A premature `foundations/ components/ patterns/ pages/` tree is empty ceremony (the same JBGE rule as progress.md → tree).

```
spec/design/                 # flat by default
├─ README.md                 # what's here + which standards + the file reference map
├─ DESIGN.md                 # the design standard: color / type / hierarchy / components (tool-free)
├─ design.tokens.json        # DTCG tokens (incl. a motion group)
├─ motion.md                 # thin global motion: principles + surface schemes + a11y; points at the tokens
├─ <component>.html          # component prototype + contract
├─ <page>.html               # page comp
└─ archive/                  # archive-aside (terminal pages; superseded docs)
```

**Graduate to subfolders only when a layer grows** — when components (or pages, or patterns) outgrow a handful of flat files, move that layer into its own `components/` (etc.) subfolder. Flat until it demonstrably isn't.

## What is actually the SOT (two kinds of design truth)

Not every `spec/design/` file is the same kind of truth:

- **`design.tokens.json` — the diffable contract, the true SOT.** Tokens are negotiated intent in diffable text: change a value, git shows the diff. Source of truth in the full sense.
- **`.html` comps — reference-grade intent, NOT pixel-truth.** A comp captures layout, hierarchy, states, and behavior *by intent*; the building agent approximates it and self-verifies by rendering + visual-diffing. It is authoritative about *what the design means*, not about exact pixels. Do not hard-code coordinates or raw values — bind to tokens.

## Content boundary — what the store holds vs points at

A ctx store holds the **diffable decision layer**, not generated or binary artifacts — the same cut that keeps compiled code out of a spec. This is the project-wide ctx content boundary (recorded as ADR 0019 in a ctx store's `decisions/`); design is its first hard case:

| inside `spec/design/` (diffable intent) | outside — pointer + intent only |
|---|---|
| `design.tokens.json` · `DESIGN.md` · which motion scheme & why | compiled CSS (generated) · `.fig` · images / logo / video / audio · tool-proprietary exports |

For anything on the far side of the line, record a **pointer + the intent** (e.g. `logo source: figma://… ; exported to code/assets/logo.svg`), never the bytes.

## Design tokens (DTCG)

A design token is a named design decision — a color, a duration, a spacing step — expressed as data, so every surface reads one source instead of hard-coding values. The interoperable format is the W3C **Design Tokens Community Group (DTCG)** JSON. The format is still evolving — **verify its current version and which token types it defines when you use it**, rather than relying on a remembered snapshot.

- Plain JSON, `.tokens`/`.tokens.json`; `$value` / `$type` / `$description` / `$deprecated`.
- **Three tiers** (via the alias mechanism `{group.token}`): primitive (`blue-500`) → semantic (`color.action`, the theming layer — don't skip it) → component (`button.primary.bg`, which references semantic, never a raw primitive).
- **Motion tokens** — `duration`, `cubicBezier`, a composite `transition`. If you need a token type the format doesn't yet define (e.g. spring presets), mark it a custom extension and check current DTCG support first.

## Motion — global tokens + LOCAL choreography (the contract)

A single monolithic `motion.md` drifts (a governance gap); no mature system uses one. Split the contract two ways:

- **GLOBAL = motion tokens** — duration scale, easing curves — in `design.tokens.json` (a `motion` group, same source as color/spacing). Under `prefers-reduced-motion`, durations collapse to ~0.
- **GLOBAL = a thin `motion.md`** — principles + named surface *schemes* + a11y rules, pointing at the tokens. Schemes are chosen at product level: `product` (subtle/fast) vs `marketing`/`expressive` (cinematic, hero moments); campaign motion lives outside the core product tokens.
- **LOCAL = choreography** — a specific hero animation, a page transition — co-located with the page/component it animates, never in the global file.

## Per-component / per-page doc contents

- **Component doc**: overview · anatomy (labeled parts) · variants · states (default/hover/focus/pressed/disabled/error) · prop/API contract (TS union variants) · token bindings (never raw hex) · a11y/keyboard · usage do/don't · one example + the `.html` prototype.
- **Page doc**: which spec behaviors the screen realizes + the hi-fi/lo-fi `.html` comp + which components/patterns it composes.
- **Reference-grade, not pixel-perfect** — this is the fidelity target we bet on (a SHOULD, not a settled law): capture intent, hierarchy, states, behavior via tokens/auto-layout; the agent approximates and self-verifies by rendering + visual-diffing the reference. Figma MCP / Code Connect output is reference-grade, not ship-grade.

## The design standard (DESIGN.md) — tool-free

`DESIGN.md` is the project's design standard: color / type / hierarchy / component conventions. It is **tool-free** — an agent authors and follows it directly, with no skill dependency. A design-system automation tool, if the project has one, can *generate or validate* `DESIGN.md` (and extract tokens/components out of a built UI); without one, hand-write it — **the standard is the same either way**. Never encode a specific tool's workspace artifacts into the spec convention.

## Archive rule (per artifact)

Follows the lifetime class (global rule in `../../ctx/references/consistency.md § Archive mechanics`):

- **components / patterns / tokens** (GLOBAL, referenced by others): edit in place for a localized change; on a foundational change, archive the whole old doc (canonical archive filename per consistency.md) and write one clean new one — **never leave a deprecated section inline**; ripple-fix references in the same commit; record why as an ADR.
- **pages** (LOCAL, terminal, nobody references): **archive-aside** into an `archive/`.
- Do not use `$deprecated`-in-place for internal docs. `$deprecated` is built for externally-distributed token/component libraries, where you can't force-update downstream consumers; here you control every reference, so fix-and-archive keeps the active docs clean.

---
name: design-system
description: How to structure spec/design/ — the design subtype of a spec, where .html and .json files are themselves the source of truth (Markdown can't express layout/color/type/motion precisely). Covers the real taxonomy (Foundations / Components / Patterns / Pages — not the invented components/flows/mockups), the DTCG 2025.10 token model, motion split into global tokens + local choreography, the Impeccable hookup (DESIGN.md / extract / animate), reference-grade fidelity, and the archive rule per artifact. Read from ctx-spec when writing any design doc.
---

# spec/design/ — Design as a Spec Subtype

Design lives close to the user's eyes: layout, color, type, spacing, motion. **Markdown can't express it precisely**, so under `spec/design/` the `.html` (hi-fi/lo-fi comps, component prototypes) and `.json` (tokens) files ARE the SOT — the one place in the KB where truth isn't Markdown.

## The taxonomy (use the real one, not an invented split)

The canonical top-level taxonomy across every mature design system (Material 3, IBM Carbon, Shopify Polaris, Atlassian, Salesforce SLDS) is:

**Foundations → Components → Patterns → Pages** (Atomic Design's top tier is literally "pages").

- **Foundations** — tokens + primitives: color, type, spacing, grid, elevation, **motion**, iconography, a11y. GLOBAL, one set per project.
- **Components** — reusable UI building blocks (button, card, modal). GLOBAL, one canonical def each.
- **Patterns** — reusable solutions to recurring problems, combining multiple components (forms, empty states, notifications). **A multi-step user flow is a pattern, not a top-level peer.**
- **Pages** — full-screen compositions built from components. LOCAL, per-feature; nobody imports a page.

> Do NOT organize as `components / flows / mockups` — that mixes axes and omits Foundations. `flows` = a pattern subtype; `mockups` = a fidelity word, use `pages/`.

## Folder structure

```
spec/design/                 # design = a spec subtype; .html/.json ARE the SOT
├─ README.md                 # what's here + which standards + the file reference map
├─ DESIGN.md                 # design-system standard (Impeccable convention): color/type/hierarchy/components
├─ foundations/              # GLOBAL layer
│  ├─ design.tokens.json     #   DTCG 2025.10; includes a motion token group
│  └─ motion.md              #   THIN global: principles + surface schemes; points at the tokens (NOT a monolith)
├─ components/               # per-component docs (.html prototype + contract). GLOBAL
├─ patterns/                 # multi-component solutions; multi-step flows live here
├─ pages/                    # full-screen hi-fi/lo-fi comps (.html). LOCAL
│  └─ archive/               #   terminal pages: archive-aside
└─ (no archive/ for components·patterns·tokens — retire via the spec rewrite rule, below)
```

## Design tokens (DTCG)

The W3C DTCG **Design Tokens Format Module reached its first stable version, 2025.10** (Oct 2025) — a Community-Group Report ("production-ready", not a formal W3C standard). Target 2025.10, not the live draft.

- Plain JSON, `.tokens`/`.tokens.json`; `$value`/`$type`/`$description`/`$deprecated`.
- **Three tiers (community convention, enabled by the alias mechanism `{group.token}`):** primitive (`blue-500`) → semantic (`color.action`, the theming layer — don't skip it) → component (`button.primary.bg`, references semantic, never a raw primitive).
- **Motion token types exist:** `duration` (§8.5, `{value,unit}`), `cubicBezier` (§8.6, 4-number array), composite `transition` (§9.5). **Caveat: no `spring` or keyframe type yet** — a Motion module is planned, not shipped. If you need spring presets, mark them as a custom extension.

## Motion — global tokens + LOCAL choreography (not one global file)

A single monolithic `motion.md` is an anti-pattern (drifts, "governance gap"). No mature system uses one. Split:

- **GLOBAL = motion TOKENS** — duration scale, easing curves, spring presets — in `design.tokens.json` (a `motion` group, same source as color/spacing). Reduced-motion: durations collapse to ~0 under `prefers-reduced-motion`.
- **GLOBAL = a THIN `foundations/motion.md`** — principles + the named surface SCHEMES + a11y rules; it points at the tokens. Schemes are chosen at product level, not baked per-token: `product` (subtle/fast — Carbon "productive", M3 "standard") vs `marketing`/`expressive` (cinematic, hero moments). Campaign motion lives OUTSIDE the core product tokens (its own stack).
- **LOCAL = choreography** — a specific hero animation, a page transition, a campaign treatment — co-located with the page/component/campaign it animates, NOT in the global file. (Carbon: microinteractions are built-in/global, "the motion design of the overarching UI … is up to each product team.")

## Per-component / per-page doc contents

- **Component doc** (converged across Storybook, shadcn/ui, Material 3, Figma): overview · anatomy (labeled parts) · variants · states (default/hover/focus/pressed/disabled/error) · prop/API contract (TS union variants) · token bindings (never raw hex) · a11y/keyboard · usage do/don't · one example + the `.html` prototype.
- **Page doc**: which spec behaviors the screen realizes + the hi-fi/lo-fi `.html` comp + which components/patterns it composes.
- **Reference-grade, not pixel-perfect** (2026 consensus): capture intent, hierarchy, states, behavior via tokens/auto-layout — the agent approximates and self-verifies by rendering + visual-diffing the reference. Do NOT hard-code coordinates or raw values. Figma MCP / Code Connect output is reference-grade, not ship-grade.

## Impeccable hookup

- **`DESIGN.md`** = the Impeccable design-system standard (color/type/hierarchy/components) — the project's current design standard.
- **`extract`** pulls reusable colors/spacing/components out of a built UI into the token system / `components/`.
- **`animate`** produces motion — its output maps to the token layer + local choreography, per the motion split above.
- On build: run a subagent to write the standards (DTCG / WAI-ARIA APG / Impeccable conventions) into `DESIGN.md` and record the file reference map in the README.

## Archive rule (per artifact)

- **components / patterns / tokens** (GLOBAL, referenced by others): retire via the spec rewrite rule — edit in place for a localized change, or on a foundational change archive the whole old doc (under the canonical archive filename in `../../ctx/references/consistency.md § Archive mechanics`) and write one clean new one; **never leave a deprecated component/section inline**; ripple-fix references same commit; record why as an ADR.
- **pages** (LOCAL, terminal, nobody references): **archive-aside** into `pages/archive/`.
- No `$deprecated`-in-place limbo for internal docs — that flag is for externally-distributed component libraries where you can't force-update downstream consumers; here we control every reference, so fix-and-archive.

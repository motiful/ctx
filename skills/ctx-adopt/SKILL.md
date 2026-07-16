---
name: ctx-adopt
description: Bring an EXISTING project (brownfield) under ctx with minimal disruption. Decides where the /ctx folder lives (configured central-root symlink when present; otherwise in-repo directory by default, or a gitignored symlink to an external sibling store named project-name-ctx when the context must not ship with public code), which pointer file announces it (respect the project's existing AGENTS.md/CLAUDE.md; else the running agent's native one), and ROUTES the actual doc-writing to the domain skills (ctx-merge / ctx-spec / ctx-progress / ctx-report) rather than writing shallow versions itself. Use when a user says "apply ctx to this existing repo / bring this repo under ctx", when deciding whether ctx must stay separable from published code, or when onboarding a messy existing repo.
license: MIT
metadata:
  author: motiful
  version: "1.2"
---

# ctx-adopt — bring an existing project under ctx, minimally

> Brownfield entry: makes ctx take effect on an existing (usually messy) project without touching what the user has — it orchestrates; the domain skills do the real writing. Lifetime model: [`../ctx`](../ctx/SKILL.md).

## The one principle: stable mount point, swappable backend

`/ctx` is a **mount point** at the project root — the fixed place any agent looks for context.
Its **backend** (where the bytes live) is swappable. The agent always reads `<repo>/ctx`;
whether that is a real folder or a symlink is a deployment detail.

## Three rules — what "low-disruption" MEANS

1. **NEVER restructure the user's existing directories.** Only ADD the `/ctx` mount + one pointer line.
2. **Honor the user's machine-level storage preference first.** If no central root is configured, default to the lightest backend (in-repo real dir). Upgrade only when separability is actually needed.
3. **Reorganize the user's tree only when the user asks** — and even then guarantee one thing: ctx logic stays valid.

## Execution Procedure

### EP1 — pick the backend

```
storage = load ctx first-run storage preference     # ../ctx/SKILL.md § First-run storage preference

if storage.central_root is set:
    backend = CENTRAL EXTERNAL SYMLINK    # machine-level preference; symlink gitignored
             <repo>/ctx  ->  <central_root>/<project-slug>-ctx/
elif project is published (LICENSE + public remote + package-registry footprint):
    backend = EXTERNAL SYMLINK           # context must not ship with public code
             <repo>/ctx  ->  ../<project>-ctx/     (symlink gitignored; real bytes live outside the code repo)
else:
    backend = IN-REPO DIR                # default, zero setup, minimal footprint
             <repo>/ctx/   (real folder; commit if the repo is private, gitignore if not ready to share)
```

Naming: the mount is `/ctx` inside the project's own directory. When the project name must be
attached to a standalone ctx artifact (the external store, or loose files sharing a parent), use
the **suffix** form `<project>-ctx` — it reads as "the ctx of `<project>`" and keeps a
project and its ctx adjacent in a directory listing.

> The external store's exact home (a sibling dir `../<project>-ctx/`, a private repo, or a shared
> machine store) is a deployment detail — use the configured central root when present; otherwise
> default to the sibling dir. Do NOT invent a central store without the first-run preference or
> an explicit user request.

### EP2 — announce it (pointer: tool-agnostic, agent-aware)

The real interop contract is the `/ctx` **folder** (vendor-neutral, like `.git`). The pointer file is only a convenience:

```
if AGENTS.md exists:    add the pointer line to AGENTS.md      # respect the project's choice
                        # @AGENTS.md BRIDGE: if the running agent is Claude Code, ALSO ensure a
                        # CLAUDE.md exists that imports it — a single line `@AGENTS.md`. Claude Code
                        # does NOT read AGENTS.md, so without this import a CC session never sees the
                        # pointer (or any other AGENTS.md instruction). One line bridges both worlds.
elif CLAUDE.md exists:  add the pointer line to CLAUDE.md
else (neither):         the RUNNING AGENT writes its own native file
                        (Claude Code -> CLAUDE.md; Codex/Cursor/Windsurf/... -> AGENTS.md)
# never force the user to switch vendors
```

Fallback: the ctx skill `description` is system-prompt-resident and already says context lives in
`/ctx`, so any agent with ctx loaded finds it even with no pointer file.
Pointer line (one line): `Project context & docs live in ./ctx — managed by the ctx skills; apply their consistency rules when editing ctx docs.`

### EP3 — sort the scattered docs (DELEGATE — do not re-implement)

Scan the existing tree for **doc-shaped** content, classify each by lifetime, and **route each to
the domain skill that owns writing it**. ctx-adopt decides *where a thing goes*; the domain skill
decides *how it is written to standard*:

| Found | Lifetime → ctx home | Delegate the WRITING to |
|---|---|---|
| scattered notes / dated research to converge | (varies) | **ctx-merge** (atomic conclusions + disposition ledger) |
| current-truth design / architecture | living → `spec/` | **ctx-spec** (normative spec + acceptance criteria) |
| a decided choice + why | append-only → `decisions/` | **ctx-spec** (ADR format + status lifecycle) |
| status / next-steps / TODO | living → `progress/` | **ctx-progress** (tree, frontmatter, handoff) |
| a doc written for a human to review | disposable → `reports/` | **ctx-report** |
| raw notes / experiments | disposable → `scratch/` | (no format contract; leave as-is) |

Produce a **migration plan** (found → ctx home → delegated skill → action), show it to the user,
**move nothing until approved.** Never touch code files. This routing is what stops ctx-adopt from
emitting shallow, standard-violating spec/decision stubs — the domain skills carry the format.

### EP4 — code-topology check (last, subordinate, respectful)

After ctx is in place, assess whether the **repo itself** would benefit from a structural change
(docs & code hopelessly interleaved; a monorepo that should split). **Surface it as a
recommendation with rationale.** Act only if (a) the user asks, or (b) building ctx made a repo
change unavoidable. Default: **respect the repo as-is.** (Absorbed repo-scaffold concern — code
topology serves ctx, never drives it; greenfield code skeletons are a one-line starter note, not a system.)

## Definition of done (stop condition = standard)

- `/ctx` mount exists at the right place with the right backend.
- pointer line added to the correct file (or skipped, with the description-fallback noted).
- **confidentiality gate:** backend holds secrets AND code repo is public → `/ctx` is a gitignored symlink out, verified never committed.
- migration plan produced (executed if approved) via the delegated domain skills; **no code files moved.**
- the ctx-folder DoD checklist (`../ctx/references/consistency.md § Rule 4 (the gate)`; skeleton model in `../ctx/SKILL.md § The canonical folder`) passes.

## Routes to
ctx (model) · ctx-merge (converge) · ctx-spec (spec/ADR) · ctx-progress (progress) · ctx-report (reports). Absorbs the former repo-scaffold.

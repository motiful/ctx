---
name: ctx-serve
description: Hosts a project's long-running processes — dev servers, watchers, build daemons — in tmux so they survive session exit and are shared across parallel agent sessions on one repo, recording their durable topology in a committed service manifest and detecting live status rather than storing it. Use when a task needs a dev server or watcher running, when parallel sessions must share one running service, when a process must outlive a /compact or session reset, or when reattaching to or restarting a service a prior session started. Records topology in ctx/services.md; a narrow operational companion in a ctx knowledge base. Not for one-shot foreground commands.
license: MIT
metadata:
  author: motiful
  version: "1.0"
---

# ctx-serve — host long-running processes across sessions

> The service manifest is a **LIVING** doc (lifetime model in [`../ctx`](../ctx/SKILL.md)). ctx records the diffable **topology** (what runs, where, how) — not the running process and not its live status; live status is **detected**, never stored.

## Execution Procedure

`ctx-serve` hosts the processes a project needs — a dev server, a file watcher, a build daemon — **detached in tmux** so one running instance survives a session exit and is **shared** by every parallel session on the repo. Its own job is small: **ensure the host · detect-before-start · start namespaced + readiness-polled · record the durable topology · reattach/restart**.

```
serve(process) → one shared instance, survivable across sessions

# STEP 0 — Ensure the host (tmux) — a precondition gate
command -v tmux ?
    present            → continue
    absent             → install with a one-line OK (macOS `brew install tmux` · Debian/Ubuntu
                         `apt-get install tmux` · …), verify `tmux -V`, continue
    can't install AND the process needs no stdin
                       → setsid headless fallback: start detached + tee a logfile + kill-by-port
                         restart (NO send-keys — setsid cannot feed stdin)

# STEP 1 — Detect before you start (live status is DETECTED, never stored)
up = lsof -ti:PORT   (or curl the endpoint)
    up   → USE it — a parallel session already hosts it; do NOT double-start
    down → start it (STEP 2)

# STEP 2 — Start detached, namespaced, readiness-polled
name = "<repo>-<service>"                      # namespaced so parallel sessions cannot collide
tmux has-session -t name ? → reuse : create    # never blind-create; never kill another session's server
tmux new-session -d -s name '<cmd, teed to a logfile>'
    #  build the argv as an ARRAY / write a temp launch script — never shell-interpolate the command
poll( lsof -ti:PORT  ||  ready-line in the logfile ) until ready or timeout
    #  new-session returning ≠ the service is up — the port is not bound yet

# STEP 3 — Interact / read
tmux send-keys -t name '<input>' Enter         # feed stdin (tmux only)
tmux capture-pane -t name -p -S -<n>           # PEEK, bounded history — a debug snapshot, NOT the log

# STEP 4 — Record the durable topology (once; and again only when it changes)
write ctx/services.md                          # 5 fields per service (below); COMMIT it
on a topology change (new service · port change):
    point the active progress leaf at ctx/services.md   # the one tracking pointer (serve ⟂ progress, below)

# STEP 5 — Stop / restart
tmux kill-session -t name    |    kill $(lsof -ti:PORT)

# Before committing — apply the cross-cutting constraints
apply("../ctx/references/consistency.md")      # same-change (services.md ↔ the launch cmd ↔ the progress pointer) · gate
```

## Why tmux (and when setsid)

tmux is the **default host** because it is a strict superset of a bare background process: it starts detached (survives the session that launched it), lets any later session **reattach** (`tmux attach` / `capture-pane`), and — uniquely — **feeds stdin** via `send-keys`. `setsid`/`nohup` also survive a session exit but **cannot feed stdin** once detached, so they are the **headless fallback** only: a dev server that just needs starting, logging, and killing, on a box with no tmux. Prefer tmux; drop to setsid only when it is absent and the process needs no input.

## The service manifest — `ctx/services.md`

A small, committed, **LIVING** file: the durable **topology** of what this project hosts. It is not progress and not live status — it is "how this project runs," the thing a new session, a new machine, or a teammate reads to bring the services up. Five fields per service:

| field | example |
|---|---|
| **name** | `myapp-web` (the namespaced tmux session name) |
| **endpoint** | `http://localhost:3000` |
| **start** | `pnpm dev` (the command STEP 2 launches, teed to the log) |
| **log** | `ctx/scratch/logs/myapp-web.log` |
| **check / restart** | `lsof -ti:3000` to detect · re-run **start** to restart |

**COMMIT it** — it is durable topology, not volatile state (the same reason a `docker-compose.yml` is committed). It changes rarely; when the topology genuinely changes (a new service, a moved port), update the manifest **in the same commit** as the change that caused it (consistency.md Rule 2). It records **topology + intent** only, never the running bytes — the content boundary a ctx store holds (diffable intent in, process/binary out).

## serve ⟂ progress — orthogonal, coupled by ONE tracking pointer

Serving and progress are **two axes**, and keeping them separate is deliberate:

| | **ctx-serve** (infrastructure axis) | **ctx-progress** (work axis) |
|---|---|---|
| tracks | what runs · ports · start/restart | where the work is · next · read-first |
| truth | `services.md` (topology, committed) + live status (**detected**, never stored) | the progress tree |
| a dev server running | = infrastructure state | ≠ a work milestone |

So do **not** merge them: a service's up/down is not progress (and it is live status → never stored); work state is not a manifest fact. The **only** coupling is a single **tracking pointer**: when a session *changes* the topology (first stands a server up, moves a port), that is a durable change this turn, so the active progress leaf records a one-line pointer to `ctx/services.md` — exactly as it points to a new ADR or an edited spec (ctx-progress's *pointer, don't restate*). On the ordinary days a service just runs, progress says nothing about it. That pointer is the whole of the "keep progress in the loop" coupling — no new mechanism.

## Live status is detected, never stored

The manifest says a service **should** be at `:3000`; whether it is up **right now** is read on demand (`lsof` / a request), never written to a file that would go stale. An agent hits the endpoint, gets connection-refused, confirms it is down, restarts per the manifest — the manifest never changes. This is why there is no "status" field: a detected truth cannot rot, a stored one can.

## Hosting hygiene (hard constraints)

> Domain MUST/NEVER for hosting a process — binding, load-before-act. The cross-cutting rules (single-source, same-change, verify-canonical, the gate) live in [`../ctx/references/consistency.md`](../ctx/references/consistency.md) and apply on top.

- **MUST poll for readiness after start.** `tmux new-session -d` (or `setsid`) returning means the process was *launched*, not that the port is bound — an immediate `lsof` check false-negatives. Poll the port or a log ready-line with a timeout before reporting "up." *(so-that: a parallel session doesn't declare the server ready and connect to nothing.)*
- **MUST namespace the session and check existence before creating.** Derive `name` from `<repo>-<service>` and `tmux has-session -t name` first. Parallel agent sessions on one repo share a tmux server; a blind `new-session` or a bare `kill-session` can collide with — or kill — another session's service. *(so-that: one session's serve action cannot start a duplicate on, or kill, another live session's server.)*
- **MUST pass the launch command safely.** Build the tmux argv as an array (no `bash -c "…<interpolated cmd>…"`); for anything with quotes/`$`/spaces, write a temp launch script and exec that. A command spliced into a shell string breaks or injects.
- **MUST read logs with a bounded history, and treat the logfile — not the pane — as the durable log.** `capture-pane -p` alone returns only the visible screen (bursts scroll off); use `-S -<n>`, and for any load-bearing decision (health, error surfacing) read the tee'd **logfile**. `capture-pane` is a debug snapshot that reflows on resize — never the source of truth for a service's output.
- **NEVER build an input-protection / "who typed this" layer.** That machinery (save-and-restore a human's in-progress input, distinguish injected vs human keystrokes) solves a *conversational agent-supervising-agent* problem. A hosted dev server has no user draft to protect and does not type back as a human — porting it is pure complexity ctx-serve does not need.

## Honest limits

tmux must be *learned* — it is not a native agent tool, which is the one real cost of hosting this way (the STEP-0 gate pays it once). `setsid` cannot feed stdin, so an interactive process on a tmux-less box is out of reach. `capture-pane` is a snapshot, not a stream — the logfile is the real record. And ctx-serve hosts processes; it does not supervise a *conversational* agent in a pane (that is a different tool with the input-protection problem this skill deliberately avoids).

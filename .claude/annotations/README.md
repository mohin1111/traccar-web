# Annotation Layer

This directory is a **read-only knowledge layer** for the source tree. It mirrors the structure of the source so each source file has a sibling `<filename>.md` describing it in LLM-friendly terms.

## Why a parallel tree?

- **Upstream-sync clean.** Upstream `traccar-web` releases regularly. If we commented inside the source files, every upstream pull would conflict in every commented file. The annotation layer lives outside the upstream paths, so `git pull upstream master` merges without friction.
- **Higher signal than line comments.** Annotations focus on the *why*, the *where it fits*, the *cross-references*, and the *gotchas* — what an LLM (or new contributor) needs and what good code can't self-document.
- **Cheap to regenerate.** When a file is heavily refactored upstream, re-read it and rewrite the annotation. No painful merge.

## Scope

This module is annotated for the **vendor-relevant subset** only — the parts Transport OS plans to lift or learn from (see parent [`CLAUDE.md`](../../CLAUDE.md)):

- `src/map/` — the MapLibre subsystem (the prime vendor target)
- `src/store/` — Redux slices (the backend data contract reference)
- `src/App.jsx`, `src/Navigation.jsx`, `src/SocketController.jsx` — app shell, routing, realtime data flow

**Not annotated:** the 28 `src/settings/` pages, 12 `src/reports/` pages, `src/login/`, `src/main/`, `src/other/` — Transport OS builds its own UI on the 14 Fleet Command Center mockups rather than forking these. App-shell helpers (`ServerProvider`, `AppThemeProvider`, `CachingController`, `reactHelper`, `ErrorBoundary`, `UpdateController`) are referenced but not individually annotated.

## Structure

```
.claude/annotations/
├── README.md          ← this file
└── src/
    ├── App.jsx.md
    ├── Navigation.jsx.md
    ├── SocketController.jsx.md
    ├── map/
    │   ├── PACKAGE.md         ← map subsystem overview
    │   ├── core/PACKAGE.md
    │   ├── control/PACKAGE.md
    │   ├── draw/PACKAGE.md
    │   ├── main/PACKAGE.md
    │   ├── overlay/PACKAGE.md
    │   └── <file>.js(x).md    ← one per source file
    └── store/
        ├── PACKAGE.md
        └── <file>.js.md
```

## Per-file annotation format (L1)

```
# <filename>

**Role:** 1-2 sentence one-liner.
**Fits in:** where in the system / who calls this.
**Read next:** [[other-file]] — cross-references.

## Public API
- `exportName` (lines X-Y) — what it exports

## Key flows
Stepwise narrative of the non-obvious flows.

## Gotchas / non-obvious
Invariants, race conditions, surprising patterns, MapLibre quirks.

## Line index
Jump-list of the lines worth knowing.
```

## Per-package annotation format (L2)

Each `PACKAGE.md` covers directory purpose, file index, dependency graph, multi-file flows, and "how to add X" recipes.

## Conventions

- Cross-references use `[[name]]` matching the annotation filename without `.md`/path.
- Line refs use `<file>:<line>` notation.
- Keep it terse — readable in under 90 seconds.
- Descriptive, not prescriptive — design decisions belong in the parent `CLAUDE.md`.

## Maintenance

When upstream changes a file significantly, regenerate the annotation rather than patching. Stale annotations are worse than none.

## Top-of-module pointers

- Parent module CLAUDE.md: [`../../CLAUDE.md`](../../CLAUDE.md)
- Top-level traccar index: [`../../../CLAUDE.md`](../../../CLAUDE.md)

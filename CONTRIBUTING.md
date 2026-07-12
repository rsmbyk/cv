# Contributing

This repo is a **static single-page CV live editor**. Almost all product behavior lives in `index.html` (markup, CSS, and one IIFE script). Keep [`README.md`](README.md) user-facing — contributor detail belongs under [`docs/`](docs/).

## Run locally

1. Clone the repo and open `index.html` in a modern browser (double-click, or serve the folder with any static server).
2. Google Fonts load from the network (`#cv-fonts`). Offline, fallbacks still render, but font picks in the Type tab need connectivity.
3. Drafts persist in `localStorage` under `cv-live-edit-v5` (plus small keys for drawer/zoom/tab). Clear site data for that origin to start fresh.

No build step, package manager, or backend.

## Before you touch live-edit or pagination

**Read the specs first.** Live content apply, `localStorage` shape, and multi-page peel/split are easy to break together.

| Topic | Doc |
| --- | --- |
| Overview, file map, edit loop | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| Page types, peel/split, continuation dots | [`docs/PAGINATION.md`](docs/PAGINATION.md) |
| Fields, lists, migrations | [`docs/CONTENT-MODEL.md`](docs/CONTENT-MODEL.md) |
| Drawer tabs, CSS vars, resets, highlights | [`docs/SETTINGS.md`](docs/SETTINGS.md) |
| Edit/Hide, zoom, print, breakpoints | [`docs/CHROME-UX.md`](docs/CHROME-UX.md) |
| Commits / what not to commit | [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md) |

Full index: [`docs/README.md`](docs/README.md).

## Pull requests

- Prefer Conventional Commits (`type(scope): subject`). See [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).
- Do **not** commit local exploration assets (`reference-*`, `*-showcase.html`, icon compare pages, etc.).
- Describe what you changed and how you verified pagination / print when those paths are involved.

# Contributing

This repo is a **static single-page CV live editor**. Almost all product behavior lives in `index.html` (markup, CSS, and one IIFE script). Keep [`README.md`](README.md) user-facing — contributor detail belongs under [`docs/`](docs/).

We use **[GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)**. See below and [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).

## Run locally

1. Clone the repo and open `index.html` in a modern browser (double-click, or serve the folder with any static server).
2. Google Fonts load from the network (`#cv-fonts`). Offline, fallbacks still render, but font picks in the Type tab need connectivity.
3. Drafts persist in `localStorage` under `cv-live-edit-v5` (plus small keys for drawer/zoom/tab, sample history, and optional Content snapshot). Clear site data for that origin to start fresh — you’ll get a random sample pack on first visit.

No build step, package manager, or backend.

## License

- App code: [`LICENSE`](LICENSE) (MIT). See also [`NOTICE`](NOTICE).
- Sample portraits: [`samples/photos/ATTRIBUTION.md`](samples/photos/ATTRIBUTION.md) (Pexels License; not MIT).

## GitHub Flow

1. **`main` is always deployable.** GitHub Pages serves from `main` `/`.
2. **Branch from latest `main`** for every change (`feat/…`, `fix/…`, `docs/…`, `chore/…`, or `hotfix/…`).
3. **Open a pull request into `main`.** Describe what changed and how you verified it (especially pagination / print / export when those paths move).
4. **Merge the PR**, then delete the branch.
5. **Do not push commits straight to `main`** for normal work — use a PR even for small docs tweaks.

### How to merge

| Kind of PR | Merge style |
| --- | --- |
| Normal work (`feat`, `fix`, `docs`, `chore`, …) | **Squash and merge** — one clean commit on `main` |
| **Hotfix** (`hotfix/…` — urgent production fix) | **Create a merge commit** — keep hotfix history visible |

Rebase merges are not part of this workflow.

### Branch naming (examples)

- `feat/content-export`
- `fix/pagination-about-split`
- `docs/github-flow`
- `chore/gitignore-samples`
- `hotfix/pages-asset-path`

## Before you touch live-edit or pagination

**Read the specs first.** Live content apply, `localStorage` shape, and multi-page peel/split are easy to break together.

| Topic | Doc |
| --- | --- |
| Overview, file map, edit loop | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| Page types, peel/split, continuation dots | [`docs/PAGINATION.md`](docs/PAGINATION.md) |
| Fields, lists, migrations | [`docs/CONTENT-MODEL.md`](docs/CONTENT-MODEL.md) |
| Drawer tabs, CSS vars, resets, highlights | [`docs/SETTINGS.md`](docs/SETTINGS.md) |
| Edit/Hide, zoom, print, breakpoints | [`docs/CHROME-UX.md`](docs/CHROME-UX.md) |
| Commits, GitHub Flow, what not to commit | [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md) |

Full index: [`docs/README.md`](docs/README.md).

## Pull requests

- Prefer Conventional Commits (`type(scope): subject`) on the branch; squash merge will fold them into one commit on `main` for normal PRs. See [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).
- Do **not** commit local exploration assets (`reference-*`, `*-showcase.html`, icon compare pages, etc.).
- Describe what you changed and how you verified pagination / print when those paths are involved.

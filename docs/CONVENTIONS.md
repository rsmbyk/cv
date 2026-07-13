# Conventions

## GitHub Flow

This project follows [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow):

- **`main`** is the single long-lived branch and is always safe to deploy (GitHub Pages → `main` `/`)
- All work happens on short-lived branches cut from `main`
- Changes reach `main` only through pull requests
- After merge, delete the feature/hotfix branch

### Merge strategy

| PR type | GitHub merge button |
| --- | --- |
| Normal (`feat`, `fix`, `docs`, `chore`, …) | **Squash and merge** |
| Hotfix (`hotfix/…`) | **Create a merge commit** |

Do not use rebase-and-merge for routine contributions.

### Branch naming

Use a type prefix and a short kebab description:

- `feat/…` — new behavior
- `fix/…` — bug fix
- `docs/…` — documentation only
- `chore/…` — tooling, ignore rules, meta
- `hotfix/…` — urgent fix against production/`main` (merge commit)

### Day-to-day

```text
git checkout main
git pull origin main
git checkout -b feat/my-change
# … commit with Conventional Commits …
git push -u origin HEAD
# open PR → main
# squash-merge (or merge-commit for hotfix) → delete branch
```

Do **not** commit directly to `main` for normal work.

## Commits

Use [Conventional Commits](https://www.conventionalcommits.org/):

```text
type(scope): imperative subject
```

Examples: `fix(cv): restore about split after drawer resize`, `docs(cv): add pagination spec`.

- **Subject:** short, imperative, plain English
- **Body:** short paragraphs — what changed and why it mattered; how only when non-obvious
- Agent-authored commits end with:

```text
Co-authored-by: Cursor <cursoragent@cursor.com>
```

On normal PRs, squash merge collapses the branch into one commit on `main` — still write clear commits on the branch so review is easy.

## Folding vs tip commits (local only)

When finishing a **docs-only** or otherwise separate slice of work on an unpushed branch:

- Prefer a **new** commit on top of the current tip
- Do **not** amend or soft-reset into an unrelated product feature commit just to keep history flat
- Folding adjacent unpushed commits is only for the **same** logical change (same topic), never across docs-vs-feature or divergent topics
- Never rewrite commits that are already on `origin`

If unsure, make a separate commit.

## What not to commit

Leave untracked (local design exploration):

- `reference.pdf`, `reference-*.png`, `reference-cv.jpg`, and similar
- `continuation-icon-showcase.html`, `icon-compare.html`, `tabler-*.html`, and other one-off compare pages

Tracked product assets stay limited to what the app needs at runtime (`index.html`, `samples.js`, `samples/photos/`, `photo.jpg`, user README, contributor docs, `LICENSE` / `NOTICE`).

## PRs

- Target **`main`**
- Describe user-visible behavior and how you tested pagination/print when those paths change
- Keep `README.md` oriented at end users, not contributors
- Squash-merge normal PRs; merge-commit hotfixes (see above)

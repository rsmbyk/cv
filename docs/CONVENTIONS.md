# Conventions

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

## Folding vs base product tip

When finishing a **docs-only** or otherwise separate slice of work:

- Prefer a **new** commit on top of the current tip
- Do **not** amend or soft-reset into an unrelated product feature commit just to keep history flat
- Folding adjacent unpushed commits is only for the **same** logical change (same topic), never across docs-vs-feature or divergent topics

If unsure, make a separate commit.

## What not to commit

Leave untracked (local design exploration):

- `reference.pdf`, `reference-*.png`, `reference-cv.jpg`, and similar
- `continuation-icon-showcase.html`, `icon-compare.html`, `tabler-*.html`, and other one-off compare pages

Tracked product assets stay limited to what the app needs at runtime (`index.html`, `photo.jpg`, user README, contributor docs).

## PRs

Describe user-visible behavior and how you tested pagination/print when those paths change. Keep `README.md` oriented at end users, not contributors.

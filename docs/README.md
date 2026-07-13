# Contributor docs

Specs for the CV Live Editor. Source of truth for behavior is [`../index.html`](../index.html) — these pages document what that file implements.

| Doc | Summary |
| --- | --- |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Static SPA model, file map, DOM regions, live-edit loop, storage, CSS variables, print vs screen |
| [PAGINATION.md](PAGINATION.md) | Page types, sidebar/main peel rules, About Me split, continuation dots, timeline `is-single`, re-paginate triggers |
| [CONTENT-MODEL.md](CONTENT-MODEL.md) | Scalars, contacts (dial codes / WhatsApp), list shapes, sample packs + reset shuffle, Content Save/Load snapshot, migrations, empty job title |
| [SETTINGS.md](SETTINGS.md) | Drawer tabs, `TYPE_DEFAULTS` / `CSS_VAR_MAP`, spacing, skill weight, resets, focus/hover highlights |
| [CHROME-UX.md](CHROME-UX.md) | Edit/Hide edge control, zoom fit, PDF/print, breakpoints, wheel block, select cursor, `document.title` |
| [CONVENTIONS.md](CONVENTIONS.md) | Commit style, folding caution, Co-authored-by, assets to leave untracked |

Start here if new: **ARCHITECTURE → PAGINATION → CONTENT-MODEL**. Settings and chrome are safer to skim until you edit those surfaces.

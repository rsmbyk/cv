# Chrome UX

Screen-only UI around the A4 preview. Hidden in `@media print`.

## Edit / Hide edge button

- Element: `#drawer-toggle` inside `.screen-chrome`
- Closed: pencil icon (`.drawer-toggle-icon--edit`), label expands to **Edit** on hover/focus
- Open (`body.drawer-open`): sidebar-collapse icon (`.drawer-toggle-icon--hide`), label **Hide**; `aria-label` / `title` swap accordingly
- `setDrawerOpen` toggles `body.drawer-open`, drawer `inert`, backdrop visibility, and `PANEL_KEY` in `localStorage`

### Stacking

| Layer | z-index |
| --- | --- |
| `.screen-chrome`, `.zoom-controls` | **1030** |
| `#drawer-backdrop` | **1040** |
| `#settings-drawer` | **1050** |
| Content reset modal | **1200** |
| Content load modal | **1200** |

**Invariant:** Edit/Hide stays **under** the drawer and backdrop for the whole open/collapse animation (commented in CSS). Do not raise chrome above 1040 or the button will poke through the drawer.

### `#drawer-close` (Close)

- Icon-only Tabler `x` with `aria-label` / `title` **Close**
- Default (wide layout): `#drawer-close { display: none }` — edge Edit/Hide is enough
- At `max-width: 900px`: shown again (`display: inline-flex`) because the drawer overlays and the edge control sits under the backdrop

## Zoom

Controls: `#zoom-out`, `#zoom-label`, `#zoom-in`, `#zoom-reset`, `#zoom-fit-height`, `#zoom-fit-width`.

- Manual steps: `ZOOM_STEPS` (0.5 … 2); Ctrl/Cmd `+` / `-` / `0` also adjust
- Fit modes: `zoomMode` `height` | `width` | `manual`
  - `fitZoomFor` uses **one** A4 page size (`pageBaseSize` on `#cv-page-1`), not the full stack
  - Available viewport subtracts body padding and `FIT_PAD` (16px)
- Applied as `--cv-zoom` on `body`; `.pages` uses `transform: scale(var(--cv-zoom))` with margin compensation via `--pages-stack-h`
- Persisted: `cv-zoom-v1`, `cv-zoom-mode-v1`
- Resize / drawer open → `scheduleFitZoom()` when mode is height or width

## PDF / print

`#export-pdf` calls `window.print()`.

Print CSS sets `print-color-adjust: exact` / `-webkit-print-color-adjust: exact` on `html`, `body`, and `.page` so sidebar background, accent triangle, and colored type survive “Save as PDF.”

Chrome, drawer, modal, and highlight overlays are `display: none`. Zoom transform is cleared.

## Breakpoints (~900px)

`@media screen and (max-width: 900px)`:

- `body.drawer-open` no longer reserves left padding for a persistent drawer strip (`padding-left: 1rem`)
- `.screen-chrome` stays at `left: 0` while open (does not slide beside the drawer)
- Backdrop becomes a real overlay (`display: block`)
- Drawer width `min(var(--drawer-w), 92vw)`
- `#drawer-close` visible (see above)

## Number-input wheel block

Focused `<input type="number">` inside the form would normally change value on wheel while scrolling the drawer. A non-passive `wheel` listener on `#settings-form` calls `preventDefault()` when the active element is that number input.

## Select pointer cursor

`.field select { cursor: pointer; }` (and matching focus styles) so Type/Spacing dropdowns feel clickable, consistent with buttons.

## `document.title` sync

`updateDocumentTitle()` (from form name lines + `[data-bind="title"]`):

| Name | Job title | `document.title` |
| --- | --- | --- |
| set | set | `{name} — {jobTitle}` |
| set | empty | `{name}` |
| empty | set | `{jobTitle}` |
| empty | empty | `CV` |

Called from `applyScalars` whenever scalars apply. Keep form as source of truth for the tab title.

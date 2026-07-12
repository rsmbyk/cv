# Architecture

## Overview

The CV Live Editor is a **static single-page app** in one HTML file. There is no framework, bundler, or server API. Opening `index.html` loads:

- A4 page preview(s) in `#cv-pages`
- Screen-only chrome (Edit/Hide, zoom, PDF)
- A settings drawer that drives live updates and persists a draft in `localStorage`

Typography tokens and layout are tuned to match an A4 Canva PDF reference (210mm × 297mm). Numeric size/padding/gap tokens are stored as numbers and applied in CSS as `calc(var(--token) * 1pt)`.

## File map

### Ships (tracked)

| Path | Role |
| --- | --- |
| `index.html` | Entire product: styles, DOM, script |
| `photo.jpg` | Default sample portrait (`DEFAULT_PHOTO`) |
| `README.md` | User-facing product description |
| `CONTRIBUTING.md` / `docs/*` | Contributor docs |

### Leave untracked (local exploration)

Do **not** commit these when they appear in the working tree:

- `reference.pdf`, `reference-*.png`, `reference-cv.jpg`, and similar mocks
- Showcase / compare pages: `continuation-icon-showcase.html`, `icon-compare.html`, `tabler-*.html`, etc.

They are design references, not runtime dependencies.

## Main DOM regions

```
body
├── .screen-chrome          # fixed Edit / Hide (#drawer-toggle)
├── .zoom-controls          # zoom + fit + PDF
├── #drawer-backdrop
├── #content-reset-modal    # Content tab reset confirm
├── #settings-drawer        # form #settings-form + tabs
├── #photo-input            # hidden file picker
└── #cv-pages.pages
    └── #cv-page-1.page     # first A4 sheet (sidebar + main)
        ├── .sidebar        # profile + side sections; ::before = accent triangle
        └── .main           # main-column sections
```

Continuation pages are created at runtime by `paginatePages()`:

- `.page.page-full-cont` — two-column chrome when the **sidebar** overflows (no profile repeat; triangle suppressed via CSS)
- `.page.page-continued` — **main-only** sheet when main content overflows

Primary sections use `[data-section="…"]`. Clones used for continued content set `data-continued="true"` and drop `id` attributes.

## Live-edit loop

Invariant: **form + in-memory `lists` / visibility / order are the source of truth**; the CV DOM is a projection that pagination may temporarily truncate (especially About Me).

Typical path:

1. **Read** — `readScalarFields()` / `readType()` / list editors update `lists`
2. **Apply** — `applyScalars` → `setContactField` / `setNameOnCv`; `applyType` → `:root` CSS vars; `renderCvLists` rebuilds list markup
3. **Persist** — `scheduleSave()` → debounced `persist()` → `localStorage.setItem(STORAGE_KEY, JSON.stringify(readForm()))`
4. **Paginate** — `schedulePaginate()` → double `requestAnimationFrame` → `paginatePages()`

Entry points:

| Trigger | Functions |
| --- | --- |
| Content scalars | `pushLive()` |
| Type / spacing controls | `pushTypeLive()` |
| Lists | `pushListsLive()` → `renderCvLists()` (which schedules paginate) |
| Section order / visibility | `applySectionOrder()` / visibility handlers → `schedulePaginate()` |

**Invariant:** After any content change that can change height, call `schedulePaginate()`. Do not permanently write truncated About text back into the form — `setAboutOnCv` / `restoreAboutFromDraft` keep the Content-tab draft whole; pagination splits only the DOM.

## `localStorage` keys

| Key | Constant | Purpose |
| --- | --- | --- |
| `cv-live-edit-v5` | `STORAGE_KEY` | Main draft JSON |
| `cv-live-edit-v4` / `v3` | (legacy only) | Loaded if v5 missing (`loadSaved`) |
| `cv-drawer-open-v1` | `PANEL_KEY` | `"1"` / `"0"` drawer open |
| `cv-zoom-v1` | `ZOOM_KEY` | Numeric zoom |
| `cv-zoom-mode-v1` | `ZOOM_MODE_KEY` | `manual` \| `height` \| `width` |
| `cv-drawer-tab-v1` | `TAB_KEY` | `content` \| `type` \| `spacing` \| `sections` |

### Draft schema (`readForm` / `persist`)

```json
{
  "fields": { "name": "…", "title": "…", "phone": "…", "email": "…", "address": "…", "linkedin": "…", "github": "…", "about": "…" },
  "photo": "photo.jpg | data:…",
  "type": { /* keys from TYPE_DEFAULTS / data-type controls */ },
  "lists": { "skills": [], "languages": [], "education": [], "experience": [], "projects": [], "references": [] },
  "contactVisibility": { "phone": true, "email": true, "address": true, "linkedin": true, "github": true },
  "sectionVisibility": { "contact": true, "about": true, "languages": true, "skills": true, "education": true, "experience": true, "projects": true, "references": true },
  "sectionOrder": {
    "sidebar": ["contact", "about", "languages"],
    "main": ["skills", "education", "experience", "projects", "references"]
  }
}
```

`name` in storage may include a literal `<br />` between two lines. Phone is stored as **local digits only** (see [CONTENT-MODEL.md](CONTENT-MODEL.md)). Type object may still contain legacy keys; `applyType` / `fillTypeForm` migrate them into current L/R pad and gap keys.

## CSS variables approach

`:root` declares design tokens (`--size-*`, `--pad-*`, `--gap-*`, `--color-*`, `--accent`, etc.). The Type and Spacing tabs bind controls with `data-type="…"`.

- `TYPE_DEFAULTS` — default control values (strings)
- `CSS_VAR_MAP` — maps those keys to CSS custom properties
- `applyType(type)` writes each present key onto `document.documentElement`

Font family uses `fontStack(name)` so `--cv-font` is a full stack, not a bare family name.

Most layout lengths are unitless in JS/CSS vars and multiplied by `1pt` in rules. Body line-height and text-align are applied as-is. Skill category weight is `--weight-skill-cat` on `.skill-cat`.

## Print vs screen

**Screen**

- Body grey canvas; pages scaled with `--cv-zoom` on `.pages`
- Drawer, Edit/Hide, zoom bar, focus/gap highlights visible
- Overflow hidden on `.page` so pagination can measure `scrollHeight` vs `clientHeight`

**Print** (`@media print`, triggered by `#export-pdf` → `window.print()`)

- `print-color-adjust: exact` (and `-webkit-`) so sidebar fill, accent triangle, and colors survive PDF export
- Chrome/drawer/modal/` .cv-*-highlight` hidden
- Zoom transform removed; each `.page` is A4 with `break-after: page`
- Columns set `overflow: visible` so clipped content is not silently cut by the screen clipper

**Invariant:** Never rely on screen-only overlays for printed layout. Pagination must leave printable DOM structure correct before print.

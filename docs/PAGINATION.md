# Pagination

Multi-page layout is implemented entirely in the script IIFE: `paginatePages`, `peelOverflowFromColumn`, `placeOverflowBlocks`, `splitAboutTextToFit`, `applySectionCont`.

Measurement uses column overflow: `measureOverflow(column)` → `scrollHeight - clientHeight > 1`. That requires `.page` / columns to clip on screen (`overflow: hidden` on `.page` and columns in the default stylesheet).

## Page types

| Class | Created by | Layout |
| --- | --- | --- |
| `#cv-page-1.page` | Static HTML | Two columns: `.sidebar` + `.main`. Profile + accent triangle live here. |
| `.page.page-full-cont` | `createFullContinuationPage()` | Same two-column grid. Used when **sidebar** content overflows. Profile is **not** cloned. `.page-full-cont .sidebar::before { display: none }` — **no triangle**. |
| `.page.page-continued` | `createContinuationPage()` | Main-only (`display: block`). Padding from `--pad-cont-main-*`. |

`isContinuationPage(column)` is true when the column’s page has either continuation class.

**Invariant:** The accent triangle (`.sidebar::before` on page 1) must remain first-page-only. Full continuation pages suppress it in CSS; do not reintroduce a triangle on clones.

## High-level algorithm (`paginatePages`)

1. `clearContinuationPages()` — reclaim primary `[data-section]` nodes from continuation pages, delete `[data-continued]` clones and extra pages, reset `display` / continuation markers on page 1.
2. Re-append sections into page-1 sidebar/main per `sectionOrder`; `applySectionVisibility()`.
3. `restoreAboutFromDraft()` — write the full Content-tab About string onto the primary `.about-text` (undo any prior DOM split).
4. If neither sidebar nor main overflows (and columns have measurable height ≥ 40px), update stack height / fit zoom and return.
5. `peelOverflowFromColumn` on overflowing columns → ordered overflow blocks.
6. **Sidebar overflow** → create `.page-full-cont` pages; `placeOverflowBlocks` with `side: "sidebar"`, `acquireNext: acquireNextFullSidebar`.
7. **Main overflow** → prefer an empty `.page-full-cont > .main` if one exists, else `.page-continued`; `acquireNextMainColumn` reuses later empty full-page mains before creating main-only pages.
8. Drop empty continuation pages.
9. Recompute `.timeline.is-single` from **full** section item counts (see below).
10. Re-apply contact visibility; re-run `setPhoneOnCv` so cloned contact rows keep WhatsApp hrefs.
11. **Do not** re-apply full About onto every `.about-text` (would undo splits).
12. `updatePagesStackHeight()`, `scheduleFitZoom()`, `syncCvFocusHighlight()`.

## Peel rules (`peelOverflowFromColumn`)

Works from the **last visible** `[data-section]` in the column backward until overflow clears (guard ≤ 400).

For that section:

1. **Has visible body items** (children of `.timeline` / `.skills-list` / `.ref-grid` / `.contact-list` / `.projects-list`):
   - Hide the last item (`display: none`), push `{ type: "item", sectionId, html }`.
   - If no visible items remain: undo item-only peels for that section, hide the whole section, push `{ type: "section", sectionId }` instead (move shell wholesale — not a continued fragment).
2. **About Me** (`data-section="about"`): see text split below.
3. **Empty / no items:** hide section, push `{ type: "section", sectionId }`.

**Invariant:** Profile (`.profile`) is **not** a `[data-section]` and is never peeled or moved.

### Sidebar vs main

- Sidebar peel feeds **full** continuation pages (two-column chrome without profile).
- Main peel fills full-page main columns first (so page 2+ can carry sidebar leftovers + main overflow), then main-only pages.
- `placeOverflowBlocks` may move whole sections or split items again when the destination column still overflows (`ensureRoom`).

## About Me text split (`splitAboutTextToFit`)

Source of truth: Content tab `[data-bind="about"]`. DOM on page 1 may show only a prefix.

Algorithm:

1. If empty or column already fits → `null`.
2. Clear text; if still overflowing → `{ moveSection: true }` (heading/shell alone does not fit).
3. Binary search max prefix length that fits.
4. Soften cut with `preferAboutBreakOffset` (prefer sentence end after ~45% of maxLen, else word boundary).
5. Return `{ fitted, remainder, moveSection: false }` or `null` if only trailing whitespace remained.

Peel then either moves the whole About section or pushes `{ type: "about-text", text: remainder }` for a continued shell. `placeOverflowBlocks` can split again on later pages.

**Invariant:** Never persist the fitted prefix into the form. Always restore full draft at the start of `paginatePages`.

## Continuation dots (`applySectionCont`)

Continued section shells may show Tabler **dots** after the section icon (“placement B”):

- `createContDotsIcon()` builds `.ico-cont.section-cont`
- `applySectionCont(heading)` wraps `:scope > svg.ico` + dots in `.icon-pair`
- `clearSectionCont` removes dots and unwraps pairs

`shouldLabelCont(column, sectionId)` decides whether a shell on a continuation page gets `continued: true`:

- Only on continuation pages
- True if the primary (or an earlier page) already showed content for that section (visible body items or non-empty About text)
- **First appearance** of a section never gets the marker

Cloned shells call `applySectionCont` only when `cloneSectionShell(…, { continued: true })`.

## Timeline `is-single`

CSS: `.timeline.is-single` hides the vertical line and entry dots (single-entry sections look cleaner).

After pagination, every `.page .timeline` is updated:

- Count children on the **primary** section’s timeline (fallback: current timeline)
- Count excludes `.is-hidden` only (items temporarily `display: none` on page 1 from peeling still count toward the full section)
- `is-single` when `fullCount <= 1`

**Invariant:** A continuation page that shows a single leftover entry from a multi-item section must **not** look like a true single-entry timeline. Base `is-single` on full section count, not “items visible on this page.”

`renderCvLists` also toggles `is-single` on education/experience when rewriting HTML (`lists.*.length <= 1`); pagination then reasserts the full-count rule across clones.

## Re-paginate triggers

`schedulePaginate()` coalesces to a double rAF. Callers include:

- `renderCvLists` / list edits
- `pushLive` / `pushTypeLive` (type and spacing change measured heights)
- Section order / visibility / contact visibility changes
- Content / sections reset
- Drawer open/close (after 300ms transition) — column width/padding changes with `body.drawer-open`
- Indirectly after photo changes via content apply paths

Also keep `updatePagesStackHeight` in sync so zoom margin (`--pages-stack-h`) matches the multi-page stack.

## Guardrails when changing pagination

- Do not break About draft restore + DOM-only split.
- Do not put the triangle on `.page-full-cont`.
- Re-run phone link repair after cloning contact HTML.
- Preserve `data-continued` vs primary section identity (`findPrimarySection` prefers `:not([data-continued])`).
- Test long sidebar, long main, long About, and combined overflow before merging.

# Settings

The settings UI is `#settings-drawer` / `#settings-form`. Tabs: `content`, `type`, `spacing`, `sections` (`DRAWER_TABS`). Active tab persists in `cv-drawer-tab-v1`.

## Drawer tabs

| Tab | Panel | Owns |
| --- | --- | --- |
| Content | `#panel-content` | Photo, scalars, contact visibility, list editors, Content reset |
| Type | `#panel-type` | Fonts, sizes, colors, icons, photo ring, triangle, timeline chrome, body align/line-height, **skill category weight** |
| Spacing | `#panel-spacing` | Pads and gaps (including L/R and continuation main pads) |
| Sections | `#panel-sections` | Section show/hide; main-column reorder; Sections reset |

Type and Spacing controls share `data-type` keys and `TYPE_DEFAULTS` / `CSS_VAR_MAP`. Per-control reset buttons are injected by `wireFieldResets()` (`[data-reset-type]`).

## `TYPE_DEFAULTS` / `CSS_VAR_MAP`

Every `data-type` key used by the form should appear in both maps (defaults as strings; CSS var names as `--…`).

Notable groups:

- **Type:** `font`, `size-*`, `color-*`, `line-height-body`, `text-align-body` (`left` \| `justify`), `weight-skill-cat` (`400` \| `500` \| `600` \| `700`), `accent`, `bg-sidebar`, `bg-main`, `photo-*`, `size-icon-*`, `triangle-deg`, `timeline-dot`, `timeline-line`
- **Spacing pads:** `pad-sidebar-{top,right,bottom,left}`, `pad-profile-{top,bottom}`, `pad-main-{top,right,bottom,left}`, `pad-cont-main-{top,right,bottom,left}`
- **Spacing gaps:** `gap-photo-name`, `gap-side-section`, `gap-main-section`, `gap-list-items` (contacts, languages & skills), `gap-timeline-items` (education, experience & projects — item gap only; projects have no timeline chrome), `gap-icon-title`, `gap-icon-contact`, `gap-title-line`, `gap-underline-content`, `gap-title-subtitle`, `gap-subtitle-content`, `gap-title-content`

`applyType` writes mapped values to `:root`. Invalid `text-align-body` or `weight-skill-cat` snap back to defaults.

Color controls use a swatch button that opens a shared custom popover (`#color-popover`) with an SV spectrum, hue slider, and a Hex | RGB toggle. Hex accepts `#RRGGBB`; RGB uses three 0–255 channel fields. Both stay in sync with the spectrum/hue and each other. A visually hidden native `<input type="color" data-type>` remains the persistence / reset source of truth. Valid hex or RGB updates that input and runs `pushTypeLive`; invalid drafts are ignored until valid, then snap back on blur. Click-outside or Escape closes the popover (Escape closes the popover before the drawer).

### Skill category weight

Control: `#t-weight-skill-cat` → `--weight-skill-cat` → `.skill-cat { font-weight: var(--weight-skill-cat) }`. Emphasizes optional category names in the Skills list.

### Legacy type keys (still migrated)

| Old | New behavior |
| --- | --- |
| `gap-heading-content` | Copies into `gap-underline-content` if missing |
| `gap-items` | Seeds `gap-timeline-items`; `gap-list-items` ≈ 0.55× if missing |
| `pad-sidebar-x` / `pad-main-x` / `pad-cont-main-x` | Seeds left and right pads |

## Scoped resets

There is **no** single “reset everything” control.

| Scope | UI | Behavior |
| --- | --- | --- |
| Content | `#reset-content` | Modal confirm → pick a Content sample pack (uniform among packs not in the last 10 shown; history in `cv-sample-history-v1`) and apply fields, lists, photo, contact visibility. **Always enabled** (not gated on `isContentAtDefaults()`). |
| Sections | `#reset-sections` | `confirm()` → visibility + order defaults. Disabled when `isSectionsAtDefaults()`. |
| Per type/spacing control | `.field-reset[data-reset-type]` | Sets that key to `TYPE_DEFAULTS[key]`, then `pushTypeLive()`. **Disabled when already at default** (`isTypeControlAtDefault`). |

Content reset does **not** touch type/spacing or section order. Sections reset does **not** touch content or type.

`syncResetButtons()` keeps type/spacing and Sections disabled states honest after edits and persist. Content reset is forced enabled.

## Focus / hover highlights

Screen-only (`.cv-focus-highlight`, `.cv-gap-highlight`). Cleared for print.

### Priority (`syncCvFocusHighlight`)

1. Focused input/select/textarea in an active settings panel
2. Else other focused mapped control in panel
3. Else hover source (`hoverCvFocusEl`)

Hover resolves whole `.field` rows and whole `.section-order-item` rows so moving between label and control does not flicker.

### Mapping

- **Content** → `resolveContentCvFocusTarget` (photo, name, binds, contact visibility, list item / subgroup → `[data-entry=…]`)
- **Type / Spacing** → `resolveTypeSpacingCvFocusTargets` by `data-type`
- **Sections** → `resolveSectionsCvFocusTargets` via `data-section-focus` / `data-section-visible`

`font` (and currently `weight-skill-cat`) intentionally map to **no** region highlight — too global / no dedicated case.

### Gap-only overlays

If `data-type` starts with `gap-` or `pad-`, `applyCvFocusForSource` calls `paintGapPadHighlights` instead of outlining whole blocks. Thin `.cv-gap-highlight` strips are positioned in page coordinates (accounts for `--cv-zoom`).

Examples: pad edges on sidebar/main/profile/continued main; vertical gaps between sections or list items; icon–title / icon–contact gutters; heading underline / meta margins.

### Sections tab hover

Hovering a `.section-order-item` (label, Show checkbox, or move buttons) highlights all visible `[data-section="{id}"]` regions across pages.

## When changing settings

- Keep `TYPE_DEFAULTS`, `CSS_VAR_MAP`, form `data-type`, and (if needed) focus/gap paint cases in sync.
- Prefer additive legacy migration over breaking old drafts.
- After token changes that affect height, ensure `pushTypeLive` → `schedulePaginate` still runs.

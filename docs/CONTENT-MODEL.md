# Content model

Content lives in the Content drawer tab and in `lists` / visibility objects in memory. The CV DOM is updated through `applyScalars`, `setContactField`, and `renderCvLists`.

## Scalars (`fields`)

Bound via `data-bind` on form controls (plus two name lines).

| Key | Form | CV target | Notes |
| --- | --- | --- | --- |
| `name` | `#f-name-line1`, `#f-name-line2` | `[data-edit="name"]` | Stored as `line1` or `line1<br />line2`. Applied with `setNameOnCv`. |
| `title` | job title input | `[data-edit="title"]` | Empty → collapse (below). |
| `phone` | phone input | `[data-edit="phone"]` | Local digits only. |
| `phoneDial` | `#f-phone-dial` (hidden ISO; dial derived) | (with phone) | Calling-code digits (no `+`), e.g. `62`. |
| `phoneCountry` | `#f-phone-dial` / dial trigger | (with phone) | ISO 3166-1 alpha-2 for the dial control (disambiguates shared dials like `+1`). |
| `email` | | `[data-edit="email"]` | `mailto:` when non-empty. |
| `address` | | `[data-edit="address"]` | Plain text. |
| `linkedin` | username | `[data-edit="linkedin"]` | Display `linkedin.com/in/{user}`; href `https://linkedin.com/in/…`. |
| `github` | username | `[data-edit="github"]` | Display `github.com/{user}`; href `https://github.com/…`. |
| `about` | textarea | `[data-edit="about"]` / `.about-text` | Full draft always; see [PAGINATION.md](PAGINATION.md). |

Photo is separate: `photo` string (`samples/photos/{id}.jpg`, fallback `photo.jpg`, or `data:` URL), not inside `fields`.

### Empty job title collapse

`syncJobTitleVisibility`:

- Empty trimmed title → `.title.is-empty`, `hidden`, `aria-hidden="true"`
- Non-empty → remove those

Keeps vertical rhythm when the role is omitted.

## Contacts

`CONTACT_KEYS`: `phone`, `email`, `address`, `linkedin`, `github`.

Each row is `[data-contact="{key}"]`. Visibility toggles use `data-visible` and `contactVisibility`; hidden rows get `.is-hidden`.

### Phone / WhatsApp

`COUNTRY_CALLING_CODES` — curated ITU list (ISO + name + dial). Custom dial control (not a native `<select>`, so flags can render on Windows): trigger and listbox show a **flagcdn** PNG (`https://flagcdn.com/w20/{iso}.png`, lowercase ISO) plus `+dial`; country name on option title / `aria-label`. Hidden `#f-phone-dial` stores the ISO. Keyboard: open with Enter/Space/Arrow keys, move with arrows, choose with Enter, dismiss with Escape / outside click.

Storage (under `fields`):

- `phone` — local digits only (e.g. `81234567890`)
- `phoneDial` — country calling digits, no `+` (e.g. `62`)
- `phoneCountry` — ISO code for the dial control (e.g. `ID`)

`normalizePhoneLocal(raw, dial)`:

1. Digits only
2. Strip leading `00…`
3. Strip leading active `phoneDial`
4. Strip leading `0` trunk digits

Display: `formatPhoneDisplay` → `+{dial} {local}`  
Href: `whatsappMeHref` → `https://wa.me/{dial}{local}` (digits only)

`setPhoneOnCv` updates **all** `[data-edit="phone"]` under `#cv-pages` (including clones after pagination).

**Default dial (no IP geolocation):** on first visit (`snapshotDefaults`), best-effort ISO from `navigator.languages` / `navigator.language` region tags, else IANA timezone → `TZ_TO_ISO`, else `62` / `ID`. Never overrides a draft that already has `phoneDial` or `phoneCountry`.

LinkedIn / GitHub usernames are stripped of URL prefixes by `normalizeLinkedInUsername` / `normalizeGitHubUsername`.

## Lists

In-memory `lists` (also under draft JSON). Editors: `#*-editor`; CV hosts: `#cv-*`.

### Languages

`string[]` — each item one language. Rendered as `<li data-entry="languages" data-entry-index="i">`.

### Skills (categories)

```ts
{ name: string; skills: string[] }[]
```

- Optional category `name`
- `formatSkillCategoryLine`: empty skills → omit row; with name → `<strong class="skill-cat">Name</strong> — a, b`; without name → comma-joined skills only
- Weight of `.skill-cat` comes from Type tab `--weight-skill-cat` (not content)

Flat string arrays are migrated (below).

### Education / experience (timeline)

```ts
{
  title, org, location,
  startYear, startMonth, endYear, endMonth, endPresent,
  desc,
  bullets?  // experience only in the editor UI
}
```

- Dates formatted by `formatEntryDates` (`Jan 2020 – Present`, etc.)
- Legacy single `date` string → `parseLegacyDate` inside `normalizeTimelineItem`
- Experience: `bullets: true` → one bullet per line in `desc` (strips leading `•- *`)
- Education: plain paragraph (no bullets toggle in UI)

Rendered as `.entry` articles with `data-entry` / `data-entry-index`.

### Projects

```ts
{
  title, reference, referenceLink, year, desc, bullets
}
```

**Year placement** (`projectEntryHtml`):

- If `reference` non-empty: year sits on the same `.entry-meta-row` as the reference
- If no reference but year set: year sits on `.entry-title-row` beside the title (avoids a blank meta line)
- `referenceLink: true` → wrap reference in `<a href>` via `normalizeHref` (adds `https://` when needed)

### References

```ts
{ name, role, phone, email }
```

Rendered in `.ref-grid` with Phone/Email labels. Reference phones are **not** run through the contact dial selector / WhatsApp helpers (free-form sample text).

## Sample defaults

### First visit (no draft)

When `loadSaved()` finds no `cv-live-edit-v5` (or legacy v4/v3) draft:

1. Build type/spacing/section defaults via `snapshotDefaults()`
2. `pickNextContentSample()` — uniform among packs **not** in the last 10 of `cv-sample-history-v1` (all 20 if history empty)
3. `applyContentSample` + `pushSampleHistory` + `persist()` so the chosen pack is the live draft

`snapshotDefaults()` / `SAMPLE_LISTS` remain only as a **fallback** when `samples.js` fails to load (Content reset or first visit).

### Content sample packs (`samples.js`)

`samples.js` exposes `window.CV_CONTENT_SAMPLES` — **20** full Content drafts. Each pack includes:

- `id`, `label`
- `fields` (name, title, phone + `phoneDial` / `phoneCountry`, email, address, linkedin, github, about)
- `lists` (skills categories, languages, education, experience, projects, references)
- `photo` — per-pack file `samples/photos/{id}.jpg` (relative path works on GitHub Pages `/cv/`)
- `contactVisibility` (e.g. hide GitHub for non-tech packs)

Packs are fictional but realistic, with distinct voices per profession. Portrait licensing: [`samples/photos/ATTRIBUTION.md`](../samples/photos/ATTRIBUTION.md) (Pexels License).

### Content reset shuffle

On **Reset to default** (after confirm modal):

1. Read recent IDs from `localStorage` key `cv-sample-history-v1` (last 10)
2. Among the **eligible** packs (`20 − last 10`; all 20 when history is empty), pick **uniformly at random**
3. Apply that pack to Content only (fields, lists, photo, contact visibility) — not type/spacing or section order
4. Append the chosen `id` to history (trim to 10)

Same pick rules as first visit. If `samples.js` fails to load, reset falls back to `defaults`.

### Content Save / Load snapshot

Separate from the live draft and sample history:

| Action | Key | Behavior |
| --- | --- | --- |
| **Save** (`#save-content`) | `cv-content-snapshot-v1` | Overwrites one snapshot: `fields`, `lists`, `photo`, `contactVisibility` |
| **Load** (`#load-content`) | same | After confirm modal, applies that snapshot to Content only; then `persist()`, paginate, sync UI. Disabled when no snapshot exists. |

Does **not** store or restore type, spacing, or section order/visibility.

Section order defaults: sidebar `contact → about → languages`; main `skills → education → experience → projects → references`. Skills stay on main; languages on sidebar (`normalizeSectionOrder` enforces this).

## Migrations

### Flat skills → categories

`normalizeListsData`: if `lists.skills` is `string[]`, wrap as `[{ name: "", skills: [...] }]`. `cloneSkillCategory` also turns a bare string into a one-skill category.

### Legacy flat fields → lists

`migrateLegacyLists(fields)` (when draft has `fields` but no `lists`):

- `skill-1` … `skill-20` → one category
- `edu-N-*` / `exp-N-*` (including legacy `*-date`) → timeline arrays
- `ref-N-*` → references
- Languages / projects fall back to samples if not present in that legacy shape

### Phone dial + local part

- Any load/apply path that sets phone runs `normalizePhoneLocal` with the active dial so older drafts with `+62…` or `08…` become local digits.
- Drafts missing `phoneDial` / `phoneCountry` migrate to Indonesia (`62` / `ID`) — the previous fixed prefix — and do **not** re-run locale detection.

### Type/spacing legacy keys

Still accepted in `type` objects (see [SETTINGS.md](SETTINGS.md)): `gap-heading-content` → `gap-underline-content`; `gap-items` → timeline/list gaps; `pad-*-x` → left/right pads.

### Storage key versions

`loadSaved` reads `cv-live-edit-v5`, else `v4`, else `v3`. Subsequent `persist` writes **v5** only.

## Empty / missing content

- Empty skill categories (no skill strings) omit `<li>` rows
- Empty job title collapses via `syncJobTitleVisibility`
- Hidden contacts/sections use `.is-hidden` (still in DOM for order editors)

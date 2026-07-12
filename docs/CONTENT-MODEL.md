# Content model

Content lives in the Content drawer tab and in `lists` / visibility objects in memory. The CV DOM is updated through `applyScalars`, `setContactField`, and `renderCvLists`.

## Scalars (`fields`)

Bound via `data-bind` on form controls (plus two name lines).

| Key | Form | CV target | Notes |
| --- | --- | --- | --- |
| `name` | `#f-name-line1`, `#f-name-line2` | `[data-edit="name"]` | Stored as `line1` or `line1<br />line2`. Applied with `setNameOnCv`. |
| `title` | job title input | `[data-edit="title"]` | Empty → collapse (below). |
| `phone` | phone input | `[data-edit="phone"]` | Local digits only. |
| `phoneDial` | `#f-phone-dial` | (with phone) | Calling-code digits (no `+`), e.g. `62`. |
| `phoneCountry` | `#f-phone-dial` | (with phone) | ISO 3166-1 alpha-2 for the selector (disambiguates shared dials like `+1`). |
| `email` | | `[data-edit="email"]` | `mailto:` when non-empty. |
| `address` | | `[data-edit="address"]` | Plain text. |
| `linkedin` | username | `[data-edit="linkedin"]` | Display `linkedin.com/in/{user}`; href `https://linkedin.com/in/…`. |
| `github` | username | `[data-edit="github"]` | Display `github.com/{user}`; href `https://github.com/…`. |
| `about` | textarea | `[data-edit="about"]` / `.about-text` | Full draft always; see [PAGINATION.md](PAGINATION.md). |

Photo is separate: `photo` string (`photo.jpg` or `data:` URL), not inside `fields`.

### Empty job title collapse

`syncJobTitleVisibility`:

- Empty trimmed title → `.title.is-empty`, `hidden`, `aria-hidden="true"`
- Non-empty → remove those

Keeps vertical rhythm when the role is omitted.

## Contacts

`CONTACT_KEYS`: `phone`, `email`, `address`, `linkedin`, `github`.

Each row is `[data-contact="{key}"]`. Visibility toggles use `data-visible` and `contactVisibility`; hidden rows get `.is-hidden`.

### Phone / WhatsApp

`COUNTRY_CALLING_CODES` — curated ITU list (name + dial). Selector label: `Indonesia (+62)`.

Storage (under `fields`):

- `phone` — local digits only (e.g. `81234567890`)
- `phoneDial` — country calling digits, no `+` (e.g. `62`)
- `phoneCountry` — ISO code for the `<select>` (e.g. `ID`)

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

`snapshotDefaults()` / `SAMPLE_LISTS` (used for first load and Content reset):

| Area | Sample |
| --- | --- |
| Name | Lorna / Alvarado |
| Title | Marketing Manager |
| Phone | `81234567890` + locale (or `62`/`ID`) dial → displays `+{dial} 81234567890` |
| Email / address | reallygreatsite placeholders |
| LinkedIn / GitHub | `username` |
| About | short Lorem paragraph |
| Skills | one unnamed category with six soft skills |
| Languages | English, Spanish, French |
| Education | two Borcelle University entries |
| Experience | four Arowwai roles (paragraph `desc`) |
| Projects | Brand Campaign Hub + GitHub-style reference + year `2023` |
| References | two people with placeholder phone/email |
| Photo | `photo.jpg` |

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

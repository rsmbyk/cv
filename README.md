# CV Live Editor

I only meant to edit my CV with an AI tool. By the end of the day, I’d built myself a fully working CV live-editor.

Open `index.html`, tweak a field, and the A4 page updates instantly. Drafts stay in your browser. Print to PDF when you’re happy. One static HTML file — no build step.

## Try it

```bash
python -m http.server 8765 --bind 127.0.0.1
```

Then open [http://127.0.0.1:8765/](http://127.0.0.1:8765/).

## Live editing

A left-hand drawer with four tabs — **Content**, **Type & icons**, **Spacing**, and **Sections** — drives the whole resume.

- Changes land on the page as you type
- Drafts auto-save to `localStorage` in this browser
- The browser tab title follows your live name and job title

Focus or hover a control and the matching region on the CV lights up. Spacing and gap controls highlight the gap strips only — not the whole block — so you see the padding or gap you’re actually changing.

## Your content

**Profile** — photo, name (two lines), and job title. Leave the title empty and it collapses cleanly.

**Contacts** — WhatsApp with a `+62` local number field, email, address, LinkedIn, and GitHub. Each contact can be shown or hidden.

**About Me** — a freeform bio that can split mid-paragraph across pages when content overflows.

**Languages** — a simple list you can grow or trim.

**Skills** — categories, one row each (category + skills on that row).

**Educations** & **Work Experiences** — timeline entries with optional bullet points under each role.

**Projects** — title, optional reference (plain text or a link), and year. The year sits beside the reference when there is one, otherwise beside the title.

**References** — people you can list as referees.

## Look & feel

**Typography** — pick a font family, then sizes and colors for name, job title, section headings, body, item titles, and subtitles. Icon sizes, photo size and ring, accent color, and the decorative triangle’s angle are all editable.

**Colors** — accent, sidebar background, main background, and photo ring.

**Paragraphs** — left or justify alignment, plus body line height.

## Spacing

Fine-grained control over where air lives on the page:

- Sidebar and main column pads — independent left/right and top/bottom
- Profile padding and photo↔name gap
- Contacts and section spacing
- List items and timeline items
- Continuation-page pads for overflow pages

## Sections

Show or hide any section. Reorder sections on the main column. Sidebar sections can only be shown or hidden — no reorder there.

## Multi-page A4

True A4 two-column layout that paginates as you write.

- Sidebar overflow continues onto full two-column pages
- Main-only continuations when the main column runs long
- A dots continuation marker on headings that spill over
- The accent triangle appears on the first page only

## Timelines

Timeline dots and line align to the section title icons. A single-item timeline hides the line. When a multi-item section continues onto another page, the timeline keeps going with it.

## Zoom & PDF

Zoom to fit height (one page in view) or fit width. Print to PDF with colors preserved — backgrounds and accents stay put.

## Resets

- **Content** — confirm in a modal; irreversible, with a clear warning
- **Sections** — restores visibility and main-column order
- **Type / Spacing** — per-control icon-only reset; disabled when already at the default

There is no global “reset everything” button.

## Edit control

A pinned left-edge control — icon-only, with “Edit” on hover. When the drawer is open it switches to a distinct Hide icon. On small screens it sits behind the drawer overlay. On large screens there’s no duplicate close button inside the drawer.

## Small UX touches

Select menus show a pointer cursor. Number fields ignore mouse-wheel so you don’t accidentally nudge values while scrolling.

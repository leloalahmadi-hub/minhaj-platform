# Minhaj Platform — منصة منهاج

**Minhaj Platform is an educational presentation-generation demo.**

منصة منهاج عرض توضيحي (Demo) لمشروع مقرر دراسي. تحوّل مادة تعليمية مرفوعة إلى مخطط عرض تقديمي منظَّم، بواجهة عربية كاملة من اليمين إلى اليسار.

> This is a course / final-project demo. It is not a production service.

---

## AI generation — how it works, and when it does not

AI generation in this project depends on the **Claude environment providing `window.claude`**. It is **not powered by a project-owned API key**, and the project contains **no backend and no API keys of any kind**.

```js
const sample = await window.claude.use("sample");
```

| Where the file is opened | AI generation | Rest of the app |
|---|---|---|
| Inside a Claude environment that provides `window.claude` | Works | Works |
| GitHub Pages, Netlify, a local `file://`, any other static host | **Disabled** | Works |

**Hosting the files on GitHub does not provide AI generation.** GitHub (or any static host) serves the HTML file only. The generation engine is supplied by the Claude environment at run time, and nothing in this repository can substitute for it.

### Degraded mode

When `window.claude` is absent the app does **not** contact any external service and does **not** show a technical error. It disables the *Create presentation* button and displays a plain Arabic message explaining that generation requires a Claude environment. Everything else stays usable:

- browsing the interface and all screens
- reopening a previously generated presentation saved in the browser
- viewing slides and navigating between them
- exporting to PPTX and JSON

---

## The four Minhaj agents

The generation pipeline follows the Minhaj agent specification. Each stage is a separate structured-JSON request, validated before the next stage runs.

1. **Educational Content Analyst** — extracts concepts, objectives and a teaching sequence from the uploaded material.
2. **Presentation Storyboard Architect** — turns the content model into slides: one idea per slide, at most four bullets.
3. **Design Agent** — proposes a consistent Arabic-first visual system (colors, typography, direction).
4. **Coordinator / Revision** — converts free Arabic feedback into structured edits and applies them to affected slides only.

The canonical project object and the validation rules (sequential slide numbering, bullet caps, `slides_count` consistency, honest export claims) are implemented in `index.html` as `blankProject()` and `validateProject()`.

---

## User journey

```
Home → Upload → Settings → AI processing → Preview → Review → Revision → Approval → Export
```

Application states: `INPUT_RECEIVED · ANALYZING_CONTENT · CONTENT_READY · BUILDING_STORYBOARD · STORYBOARD_READY · DESIGNING · PREVIEW_READY · AWAITING_APPROVAL · REVISION_REQUESTED · FINALIZING · EXPORT_READY · COMPLETED · ERROR`

---

## Running it

No build step, no package manager, no dependencies to install.

```bash
# any static server, or simply open index.html
python3 -m http.server 8000
```

Then open `http://localhost:8000`. AI generation stays disabled outside a Claude environment — see the table above.

---

## Supported input formats

| Format | Notes |
|---|---|
| `.txt`, `.md` | Read directly in the browser |
| `.pdf` | Text extraction via pdf.js loaded from a CDN. If the library does not load, `.pdf` is removed from the advertised formats at run time and the user is directed to paste text instead. |
| Pasted text | Always available |

Scanned (image-only) PDFs are not supported — there is no OCR.

---

## Export

Both export paths run entirely in the browser. **No export API, no upload, no server.**

- **PPTX** — an OOXML writer and a store-only ZIP writer (CRC32) implemented inside `index.html`. No external library. The output was verified to open correctly with `python-pptx` (16:9, Arabic RTL text, bulleted lists).
- **JSON** — the full Minhaj project object.

Delivery: inside Claude the `downloads` capability handles the save and keeps the Arabic file name; on a plain static host an ordinary `<a download>` is used, and the file is named `minhaj-presentation.pptx` / `.json` — Chromium discards a non-ASCII name on a `blob:` URL. Neither path makes a network request.

**Not implemented:** Canva integration and PDF export. The interface says so explicitly and never renders an "Open in Canva" button or a fabricated link.

---

## Privacy

- Uploaded material is read in the browser. It is sent only to the Claude environment as part of a generation request, and only when the user starts generation.
- A copy of the current presentation is kept in the browser's `localStorage` so work survives a page reload. It never leaves the device and is cleared by *Start a new presentation*.
- No analytics, no tracking, no cookies, no third-party services beyond the two asset hosts below.

## External resources

Only static assets, no credentials, no data sent:

- `fonts.googleapis.com` / `fonts.gstatic.com` — IBM Plex Sans Arabic webfont
- `cdnjs.cloudflare.com` — pdf.js, used solely to read PDF text locally

The `schemas.openxmlformats.org` URLs in the source are XML namespace identifiers used by the PPTX writer. They are never fetched.

---

## Security

This repository contains **no API keys, no tokens, no secrets and no credentials**, and the application makes **no `fetch`, `XMLHttpRequest` or `WebSocket` calls** of its own.

Do not add an API key to this project. `index.html` is downloaded in full by every visitor, so any key placed in it would be readable by anyone.

---

## Tech stack

Static HTML + vanilla JavaScript. No framework, no build tooling, no dependencies. One file: `index.html`.

## Project structure

```
.
├── index.html      # the entire application
├── README.md
└── .gitignore
```

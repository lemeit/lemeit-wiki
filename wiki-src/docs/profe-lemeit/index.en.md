# ![Profe](../assets/logos/profe.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Profe Lamaita — profe.lemeit.ar

Personal site of **Prof. Eng. Luciano Lamaita**, physics teacher in Saladillo, Buenos Aires. Brings together the Spanish translation and adaptation of Jeff Bigler's notes, class notes from the 4 institutions where he teaches, and the technical projects (EMA, Eureka AI, DVBA GIS).

- **Main site:** [profe.lemeit.ar](https://profe.lemeit.ar)
- **Concept map:** [profe.lemeit.ar/conceptos](https://profe.lemeit.ar/conceptos) — Obsidian-style note graph
- **Repository:** [github.com/lemeit/aboutme](https://github.com/lemeit/aboutme)

## Architecture — two generators in one repo

```
aboutme/ (single repo)
├── content/               → Hugo (portal, includes /notes)
├── quartz/                → standalone Quartz v5 (concept map)
└── Cloudflare Pages       → a single deploy, both serve under profe.lemeit.ar
```

The main portal uses **Hugo** with the **PaperMod** theme; the concept map (`/conceptos`) is a separate **Quartz v5** site living in the same repo's `quartz/` subdirectory, built for Obsidian-style notes with a link graph. Both share a color palette (synced manually in each one's CSS) so the transition between `/notes` and `/conceptos` feels like one site, not two products taped together.

Deploy: every push to `main` triggers an automatic build on Cloudflare Pages (~1-2 min) — no manual steps, unlike `lemeit-emas` (which needs `wrangler pages deploy`).

## Stack

| Component | Technology |
|---|---|
| Static generator (portal) | [Hugo](https://gohugo.io/) v0.163+ with the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme |
| Concept map | [Quartz v5](https://quartz.jzhao.xyz/) (Obsidian format) |
| Deploy | Cloudflare Pages — auto-deploy on push to `main` |
| Locale | `es-AR` (Hugo) / `es-ES` (Quartz), dates `DD/MM/YYYY` |

## Portal content (`content/`)

| Section | What it is |
|---|---|
| `notes/notas-fisica/` | Spanish translation and adaptation of Jeff Bigler's *Physics 1: Mechanics in Plain English* (Lynn English High School), organized by chapter |
| `notes/herramientas/` | Digital tool guides — currently covers Tracker (video analysis for uniform motion, uniformly accelerated motion, projectile motion) |
| `notes/fisica-4to/`, `fisica-5to/`, `fisica-6to/` | Course notes by year, Instituto Niño Jesús and Colegio Madre Teresa |
| `notes/lab-industrias-7mo/` | Industrial Laboratory, 7th-year Chemical Technician track, EEST N°1 "Gral. Savio" — the course that all of the Environmental Monitoring Network originated from |
| `projects/` | Technical projects: EMA Saladillo, Eureka AI, DVBA GIS |
| `propuestas/` | Institutional teaching proposals |

### Progress on the Bigler translation (`notas-fisica/`)

| Chapter | Status |
|---|---|
| 01 · Lab | ✅ complete (9 notes) |
| 02 · Math | ✅ complete (6 notes) |
| 03 · 1D Kinematics | ✅ complete (10 notes) |
| 04 · 2D Kinematics | 🔄 in progress |
| 05 · 1D Forces | ⬜ in preparation |
| 06 · 2D Forces | ⬜ pending |

Translation, pedagogical adaptation and integration with field activities by Luciano Lamaita, with the original author's authorization (license [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)).

## PDF guides (paper format)

Any note on the portal can be turned into a PDF formatted like an academic paper (two columns, Georgia typeface, APA style), with the KaTeX formulas already rendered. The PDFs live in `static/files/pdf/` and are linked from the note itself.

Pieces of the mechanism:

- **`assets/css/extended/print-paper.css`**: CSS that only activates under `@media print`. Defines an A4 page with 20mm×14mm margins, 9.8pt Georgia, two text columns, APA-style tables (3 horizontal rules, no vertical grid), h2/h3 headings with thin rules, and a footer with the page number. Doesn't affect the on-screen view — `custom.css` handles that.
- **`scripts/generate-pdf.mjs`**: a Node.js script using Playwright (headless Chromium). It looks for `.md` files with `pdf = true` in the frontmatter, spins up a local HTTP server over `./public/` (Hugo generates absolute CSS/JS paths that don't resolve over `file://`), and for each note it opens the page, activates `@media print`, waits for KaTeX to finish rendering, injects the byline (`Luciano Lamaita · profe.lemeit.ar · date`), expands any table wider than the column to `column-span: all`, and calls `page.pdf()` (A4, with page number/total), saving the result to `static/files/pdf/<slug>.pdf`.
- **`layouts/partials/extend_head.html`**: loads KaTeX from a CDN only if the note has `math = true`. It's the very same KaTeX rendered on screen that Playwright renders before printing, so the PDF's formulas come out identical to the web version's.
- **`resource-box`** in the `.md` file: an HTML block with a `no-print` class (doesn't appear in the PDF) with the download link:

```html
<div class="resource-box no-print">

Resources

📄 <a href="/files/pdf/nombre-nota.pdf">Download as PDF (paper format)</a>

</div>
```

### Adding a PDF to a new note

**1. Frontmatter**

```toml
+++
title = 'Note title'
math = true
pdf = true
+++
```

**2. `resource-box` at the start of the content**

```html
<div class="resource-box no-print">

Resources

📄 <a href="https://profe.lemeit.ar/files/pdf/nombre-nota.pdf" target="_blank" rel="noopener">Download as PDF (paper format)</a>

</div>
```

The PDF's slug is the note's path relative to `content/`, without `.md`. For example, `content/notes/notas-fisica/cap03-cinematica-1d/mruv.md` generates `static/files/pdf/notes/notas-fisica/cap03-cinematica-1d/mruv.pdf`.

**3. Generate the PDF**

```powershell
cd C:\GitHub\aboutme
hugo                          # generates ./public/
node scripts/generate-pdf.mjs # generates the PDFs in static/files/pdf/
```

Requires Node.js and Playwright installed:

```powershell
npm install playwright
npx playwright install chromium
```

**4. Commit it all together**

```powershell
git add content/notes/path/note.md
git add static/files/pdf/path/note.pdf
git commit -m "note: add PDF for <title>"
git push
```

### Formulas that don't fit in one column

If a formula is wider than the print column (~83mm), it doesn't shrink automatically — it has to be split in the `.md` using `\begin{aligned}...\end{aligned}`:

```latex
$$
\begin{aligned}
  w_{real} &= \frac{w_{ideal}}{\eta} \\
            &= \frac{0.100}{0.72} = 0.139\ \tfrac{\text{kJ}}{\text{kg}}
\end{aligned}
$$
```

Wide tables do expand automatically to both columns — the script detects overflow and applies `column-span: all`.

## The concept map (`/conceptos`)

A separate Quartz v5 site, with notes in Obsidian format, organized into 7 thematic areas: Mechanics, Gravitation, Matter, Thermodynamics, Waves, Electricity and Magnetism, Modern Physics. The root `index.md` has explicit wikilinks to all 7 areas so the graph shows the connection network right from the home page, and the side Explorer follows that same thematic order (not alphabetical) via a custom `sortFn` in `quartz.ts`. Local graph with `depth: 2` (from one note you see its direct neighbors and its neighbors' neighbors); a global graph is available via a toggle.

## Design — the "X-Wing Poe Dameron" palette

Warm colors (cream + burnt orange) shared between Hugo and Quartz, with dark mode synced manually between both generators:

| Variable | Value (light mode) | Use |
|---|---|---|
| `--primary` | `#9B3D00` | Links, headings |
| `--secondary` | `#5C2200` | Secondary text, hover |
| `--theme` / `--entry` | `#FAF7F2` | Warm cream background |

In dark mode, the critical selector is `:root[data-theme="dark"]` (not `.dark`) to match PaperMod v8+'s specificity; links need `color: #B84800 !important` to beat the theme's own selectors.

## Institutions where the courses are taught

| Course | Institution |
|---|---|
| Intro to Physics — 4th year | Colegio Madre Teresa + Instituto Niño Jesús (INJ) |
| Physics — 5th year | Instituto Niño Jesús (INJ) |
| Classical and Modern Physics — 6th year | Instituto Niño Jesús (INJ) |
| Industrial Laboratory — 7th TQ | EEST N°1 "Gral. Savio" |

## Implementation notes

- **Two generators, one repo**: keeping the palette synced manually between `assets/css/extended/custom.css` (Hugo) and `quartz/quartz.config.yaml` (Quartz) is this project's main source of manual work — there's no shared design system like `design.lemeit.ar` in the Environmental Network, because Hugo and Quartz don't share a runtime.
- **Deterministic section order**: every `_index.md` under `notes/` has a unique `weight` (1–6); without that, Hugo orders sections unpredictably.
- **`public/` isn't committed**: Hugo's output folder is in `.gitignore` — Cloudflare Pages generates it on every deploy, same as `site/` in this wiki.

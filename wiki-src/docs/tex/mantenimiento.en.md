# Maintenance and deployment

Updated 6 November 2026.

## Independent sources

- Application: private `lemeit/lemeit-tex` repository, served from `site/`.
- Public documentation: `lemeit/lemeit-wiki`, source under `wiki-src/docs/tex/`.
- Profe Lamaita: `lemeit/aboutme`; its old `apps/tex` folder is not the TeX deployment source.

The application uses neither Hugo nor Quartz. PDFs generated for Profe's notes
use a separate workflow from documents compiled by TeX users.

## Browser compilation

HTML/CSS build the interface; JavaScript controls editing and projects.
The precise mode uses **WasmTex/pdfTeX, TeX Live 2025**, compiled to WebAssembly
and running in a browser Web Worker. PDF.js renders pages and their text layer;
SyncTeX relates positions to real source files and lines.

Disabling SyncTeX retains **SwiftLaTeX/pdfTeX 2020** compatibility mode without
source mapping. Both engines have separate resources and package caches. The
precise engine, format and mirror must remain a compatible set.

Cloudflare serves the application and a restricted GET/HEAD bridge to public
files from a fixed TeX Live mirror. It does not receive documents to compile them,
accept arbitrary destinations or store student projects. IndexedDB stores projects
locally. There are no accounts, collaborative editing or automatic document sync.

## From GitHub to the live site

| Step | System | Action |
| --- | --- | --- |
| PR targeting `main` | GitHub Actions | Installs dependencies and runs `npm run deploy:check` |
| Authorized merge | GitHub | Updates `main` |
| Production build | Cloudflare Workers Builds | Takes `main` and runs `npm run deploy` |
| Public site | `lemeit-tex` Worker | Serves `tex.lemeit.ar` |

Workers Builds uses root directory `/`, no separate build command, and deployment
command `npm run deploy`, which validates again before publishing. GitHub Actions
does not deploy by itself; the Cloudflare integration does when `main` changes.
**Merging or pushing to `main` may publish**, so authorization is required.
Check the build result and live site.

Tokens belong in Cloudflare configuration, never in Git or documentation.
An authorized manual `npm run deploy` remains available as an alternative.

## Checks and cache versions

In the TeX repository:

```sh
npm ci
npm run validate
npm run deploy:check
```

`deploy:check` packages without deploying. Also verify affected browser workflows,
identity and relevant screen sizes.

For interface changes, update `SHELL` in `site/sw.js`, its expectations in
`tests/offline.test.cjs` and versioned resource URLs. Do not clear projects or
package caches to refresh the interface. Do not rebuild logos or guides routinely.

The wiki has its own check:

```sh
cd wiki-src
mkdocs build --strict --site-dir ../site-check
```

Never commit generated output. Cloudflare Pages builds and publishes the wiki
from its own `main`; wiki edits do not alter the application.

## Update record

| `lemeit-tex` PR | Change |
| --- | --- |
| #1 | Precise engine, SyncTeX and menus above floating panels |
| #2 | Three views, assisted editing and appearance gallery |
| #3 | Zoom, fit width and fit page |
| #4 | Search above dividers, scrollbars and panels |

Published after #4: `main` at `d610019`, interface cache
`lemeit-tex-shell-v64-busqueda`, 71 tests and packaging checked.
Alternative names and designs remain proposals until explicitly approved; the
published product continues to be **TeX**.

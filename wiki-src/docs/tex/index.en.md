# TeX · LaTeX documents

TeX is Lemeit's browser-based LaTeX editor, combining an editor, PDF preview, starter templates and locally stored projects. Its interface is in Spanish and the project is **under development**. Updated 6 November 2026.

## Links

- [Open the editor](https://tex.lemeit.ar/).
- [Spanish PDF guide](https://tex.lemeit.ar/guia-tex.pdf) and [editable guide ZIP](https://tex.lemeit.ar/guia-tex.zip).
- [Project presentation](https://profe.lemeit.ar/en/projects/tex/).
- [Independent lemeit-tex repository (restricted access)](https://github.com/lemeit/lemeit-tex).

## Current features

- Code, split and PDF-only views, with adjustable panel sizes.
- Local syntax highlighting, command suggestions with Tab, bracket completion and indentation.
- File, Edit, Insert, Format, View and Help menus (labels currently in Spanish).
- Find/replace, go to line, comment selection and jump to LaTeX errors.
- SyncTeX: select PDF text or Ctrl/Command + click to locate the source; Alt + F9 goes back to the PDF.
- Percentage zoom, fit width and fit page; separate appearance schemes for code, console and PDF.
- Floating or docked, resizable help and activity panels.

The application, templates and guide remain in Spanish. An English application version is planned, not available yet. This wiki provides English documentation.

See [usage, templates and headers](uso-y-plantillas.md) and [maintenance and deployment](mantenimiento.md).

## Workflow

Choose a template, start blank or import a compatible ZIP. Edit and compile manually or automatically, inspect the PDF, and consult the integrated help and compilation log. Export the PDF for distribution and a ZIP or `.tex.json` backup to keep editing later. Multi-file projects support selection of the main document.

## Local storage and offline use

Projects and resources are stored in the browser's IndexedDB. They do not sync between devices. Clearing browser data may delete documents, so keep exported backups.

Offline controls are under Edición → Preferencias → Uso sin conexión. The first visit requires internet access. Offline preparation caches the application and resources used by the prepared project. The cache check compiles without contacting the package mirror. Packages not yet downloaded still require a connection, and browsers may clear stored resources.

## Architecture and limits

The LaTeX engine runs in the browser, without a compilation server. The application is a separate static site deployed through Cloudflare Workers Static Assets, independently of Hugo, Quartz and this wiki.

Scripts, Biber, XeLaTeX and external programs are not executed. Compatibility depends on the engine and available packages. ZIP import limits documented in development are 42 MB compressed, 300 accepted files and 50 MB unpacked, with 10 MB per file and 1 MB per text file.

Dependency licenses and attribution are recorded in `site/vendor/NOTICE.md`; components do not share a single assumed license.

## Identity

The geometric TeX mark uses orange `#9B3D00` and teal `#009688`. It identifies a Lemeit tool and is distinct from Profe Lamaita's handwritten personal signature.

## Code, checks and deployment

The application lives in `lemeit/lemeit-tex`, independently of `aboutme`. GitHub stores code and runs validation and packaging checks through Actions. Cloudflare serves `site/` using the existing `lemeit-tex` resource. The browser compiles with WasmTex/pdfTeX 2025 by default (SyncTeX), or SwiftLaTeX in compatibility mode. PDF.js displays the result. A restricted Worker serves public files from a fixed TeX Live mirror; it does not receive documents for compilation.

GitHub Actions validates without deploying itself. **Cloudflare Workers Builds is connected to this repository and its `main` branch**: merging or updating that branch triggers deployment using `npm run deploy`, which validates again. Publishing requires the owner's authorization. See [the deployment workflow](mantenimiento.md).

As of 6 November 2026, PRs #1 through #4 integrate the view modes, assisted editing, appearance, zoom and search fixes into `main`. Validation passed 71 tests and packaging checks; interface cache: `lemeit-tex-shell-v64-busqueda`.

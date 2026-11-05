# TeX · LaTeX documents

TeX is Lemeit's browser-based LaTeX editor, combining an editor, PDF preview, starter templates and locally stored projects. Its interface is in Spanish and the project is **under development**. Updated 5 November 2026.

## Links

- [Open the editor](https://tex.lemeit.ar/).
- [Spanish PDF guide](https://tex.lemeit.ar/guia-tex.pdf) and [editable guide ZIP](https://tex.lemeit.ar/guia-tex.zip).
- [Project presentation](https://profe.lemeit.ar/en/projects/tex/).
- [Independent lemeit-tex repository (restricted access)](https://github.com/lemeit/lemeit-tex).

## Workflow

Choose a template, start blank or import a compatible ZIP. Edit and compile manually or automatically, inspect the PDF, and consult the integrated help and compilation log. Export the PDF for distribution and a ZIP or `.tex.json` backup to keep editing later. Multi-file projects support selection of the main document.

## Local storage and offline use

Projects and resources are stored in the browser's IndexedDB. They do not sync between devices. Clearing browser data may delete documents, so keep exported backups.

The first visit requires internet access. Offline preparation caches the application and resources used by the prepared project. The cache check compiles without contacting the package mirror. Packages not yet downloaded still require a connection, and browsers may clear stored resources.

## Architecture and limits

The LaTeX engine runs in the browser, without a compilation server. The application is a separate static site deployed through Cloudflare Workers Static Assets, independently of Hugo, Quartz and this wiki.

Scripts, Biber, XeLaTeX and external programs are not executed. Compatibility depends on the engine and available packages. ZIP import limits documented in development are 42 MB compressed, 300 accepted files and 50 MB unpacked, with 10 MB per file and 1 MB per text file.

Dependency licenses and attribution are recorded in `site/vendor/NOTICE.md`; components do not share a single assumed license.

## Identity

The geometric TeX mark uses orange `#9B3D00` and teal `#009688`. It identifies a Lemeit tool and is distinct from Profe Lamaita's handwritten personal signature.

## Code, checks and deployment

The application lives in `lemeit/lemeit-tex`, independently of `aboutme`. GitHub stores code and runs validation and packaging checks through Actions. Cloudflare serves `site/` using the existing `lemeit-tex` resource. The browser compiles LaTeX with SwiftLaTeX/pdfTeX via WebAssembly; PDF.js displays the result.

GitHub checks **do not deploy**. Automatic deployment from GitHub to Cloudflare is not connected: publishing remains an explicit action. Use `npm run validate`, then `npm run deploy:check` for a non-publishing packaging check, and, with authorization, `npm run deploy` to update the live site.

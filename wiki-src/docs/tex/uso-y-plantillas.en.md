# Usage, templates and headers

Updated 6 November 2026. Application labels are currently in Spanish.

## Start a document and keep backups

**Archivo → Elegir plantilla** offers a blank document, guided reports, assignments
and monographs. The portal layout includes **Apunte en columnas** and **Clase y
actividades**. Cover, color, font, column and header options depend on the model;
publisher templates retain their own requirements.

The document name beside the menus determines download filenames. It does not
change the printed title, controlled by `\title` or `\LemeitTitulo`.

| Export | What it keeps |
| --- | --- |
| PDF | Final reading, printing or submission copy |
| Project ZIP | Source, images and other project files |
| TeX copy (`.tex.json`) | Project plus active and main file selections |
| Current file | Only the file currently open in the editor |

Automatic saving is local to this browser, without accounts or device sync.
Export a ZIP or TeX copy to keep editing elsewhere. Loading another template keeps
a recoverable previous-project copy.

## Editing and reading

**Código / Dividida / Vista previa** switches between code, split and PDF-only
layouts. Drag the split divider to resize. PDF controls provide **− / +**, a
25–300 % field, **Ancho** (fit width) and **Página** (fit page). Initially the viewer
fits width in split view and the full page in PDF-only view. Explicit choices are
remembered locally. Zoom does not change the downloaded PDF.

**Edición → Preferencias** provides independent visual schemes for the editor,
console and PDF. Sepia and dark PDF reading modes also affect images on screen;
use Original when checking exact colors.

Suggestions appear for commands such as `\sec`: arrows select, **Tab** inserts
and **Esc** dismisses. Enter remains a new line. Syntax colors, suggestions, bracket
completion and indentation can be disabled. Suggestions are a basic catalog and
highlighting is not a complete LaTeX syntax validator.

| Action | Shortcut |
| --- | --- |
| Find / replace | Ctrl or Command + F / H inside the editor |
| Go to line | Ctrl + Alt + G |
| Comment / uncomment | Ctrl or Command + / |
| Update preview | Ctrl or Command + Enter |
| Save a project copy | Ctrl or Command + S |
| Jump to a LaTeX error | Alt + F8 |
| Source → PDF | Alt + F9 |
| Help | F1 |
| Activate menus | Alt or F10; arrows and Enter; Esc returns |

With SyncTeX enabled, select PDF text or Ctrl/Command + click to locate the real
source line and file. After editing, wait for the new PDF. A match may cover a
paragraph or equation environment rather than individual characters.

Help and console panels can float, resize or dock at an edge. Drag their headers
to move them. Find/replace stays above dividers and panels without blocking the
editor; Esc closes it and returns focus to the source.

## Template and logo sources

Within the private `lemeit/lemeit-tex` repository:

| Path | Purpose |
| --- | --- |
| `site/templates/` | Template files and resources |
| `site/templates.js`, `site/template-builder.js` | Catalog and model construction |
| `site/lemeit-template.js` | Institutional profiles and header layout |
| `site/templates/lemeit/` | School, subject and guide logos |

School crests are `madre-teresa.png` and `eest1.png`; subject logos are
`intro-fisica.png` and `lab-industrias.png`. Creating a project copies the selected
resources as `logo-institucion.png` and `logo-materia.png`. The custom profile does
not automatically include logos. Institutional trademarks retain their rights.

Editing a document does not change its original model. Publishing a changed model
affects future documents, without rewriting users' saved projects.

## Editing an existing header

Near the start of the source, change fields such as:

```latex
\newcommand{\LemeitInstitucion}{EEST N°1 Gral. Savio}
\newcommand{\LemeitLocalidad}{Saladillo}
\newcommand{\LemeitMateria}{Laboratorio de Industrias}
\newcommand{\LemeitCurso}{7° TQ}
```

`\LemeitCabecera` defines the layout. `minipage` groups items, `\hspace` controls
horizontal space and `\includegraphics` sets image dimensions. To lower only the
school logo by 2 mm:

```latex
\raisebox{-2mm}{%
  \includegraphics[width=9mm,height=12mm,keepaspectratio]{logo-institucion.png}%
}
```

A positive value raises the logo; a negative value lowers it. The required value
depends on transparent image padding and the text block. For matching school and
town typography, replace the old text block, including its gray `\scriptsize`
town line, with:

```latex
\raggedright
\fontsize{9}{10.5}\selectfont
\color{black}
\textbf{\LemeitInstitucion}\\
\textbf{\LemeitLocalidad}
```

The PDF is a preview, not a drag-to-edit canvas. Insert another image and reference
its filename, or replace the image inside an exported ZIP using the same name,
then import it again.

## External templates and offline work

Import the complete ZIP: pasting a `.tex` file does not bring classes, images or
bibliography files. Compatible pdfLaTeX and BibTeX projects are supported; Biber,
scripts and external programs are not executed. Some templates need another engine.

Under **Edición → Preferencias → Uso sin conexión**, prepare the project and check
its stored resources. The check uses a fresh compiler with cache-only access.
New templates may require additional packages; the whole TeX Live distribution is
not included, and the browser may evict stored resources. Keep exported backups.

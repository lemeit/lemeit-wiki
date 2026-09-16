# ![Profe](../assets/logos/profe.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Profe Lamaita — profe.lemeit.ar

Sitio personal del **Prof. Ing. Luciano Lamaita**, docente de Física en Saladillo, Buenos Aires. Reúne la traducción y adaptación al español de las notas de Jeff Bigler, apuntes de clase de las 4 instituciones donde dicta materias, y los proyectos técnicos (EMA, Eureka AI, DVBA GIS).

- **Sitio principal:** [profe.lemeit.ar](https://profe.lemeit.ar)
- **Mapa de conceptos:** [profe.lemeit.ar/conceptos](https://profe.lemeit.ar/conceptos) — grafo de notas estilo Obsidian
- **Repositorio:** [github.com/lemeit/aboutme](https://github.com/lemeit/aboutme)

## Arquitectura — dos generadores en un solo repo

```
aboutme/ (repo único)
├── content/               → Hugo (portal, incluye /notes)
├── quartz/                → Quartz v5 independiente (mapa de conceptos)
└── Cloudflare Pages       → un solo deploy, ambos sirven bajo profe.lemeit.ar
```

El portal principal usa **Hugo** con el tema **PaperMod**; el mapa de conceptos (`/conceptos`) es un sitio **Quartz v5** aparte que vive en el subdirectorio `quartz/` del mismo repo, pensado para notas estilo Obsidian con grafo de enlaces. Ambos comparten paleta de colores (sincronizada a mano en el CSS de cada uno) para que la transición entre `/notes` y `/conceptos` se sienta como un solo sitio y no dos productos pegados con cinta.

Deploy: cada push a `main` dispara un build automático en Cloudflare Pages (~1-2 min) — sin pasos manuales, a diferencia de `lemeit-emas` (que necesita `wrangler pages deploy`).

## Stack

| Componente | Tecnología |
|---|---|
| Generador estático (portal) | [Hugo](https://gohugo.io/) v0.163+ con tema [PaperMod](https://github.com/adityatelange/hugo-PaperMod) |
| Mapa de conceptos | [Quartz v5](https://quartz.jzhao.xyz/) (formato Obsidian) |
| Deploy | Cloudflare Pages — auto-deploy en push a `main` |
| Locale | `es-AR` (Hugo) / `es-ES` (Quartz), fechas `DD/MM/YYYY` |

## Contenido del portal (`content/`)

| Sección | Qué es |
|---|---|
| `notes/notas-fisica/` | Traducción y adaptación de *Physics 1: Mechanics in Plain English* de Jeff Bigler (Lynn English High School), organizada por capítulo |
| `notes/herramientas/` | Guías de herramientas digitales — hoy cubre Tracker (análisis de video para MRU, MRUV, tiro oblicuo) |
| `notes/fisica-4to/`, `fisica-5to/`, `fisica-6to/` | Apuntes de cátedra por año, Instituto Niño Jesús y Colegio Madre Teresa |
| `notes/lab-industrias-7mo/` | Laboratorio de Industrias, 7° Año Técnico Químico, EEST N°1 "Gral. Savio" — la materia de origen de toda la Red Ambiental |
| `projects/` | Proyectos técnicos: EMA Saladillo, Eureka AI, DVBA GIS |
| `propuestas/` | Proyectos pedagógicos institucionales |

### Avance de la traducción de Bigler (`notas-fisica/`)

| Capítulo | Estado |
|---|---|
| 01 · Laboratorio | ✅ completo (9 notas) |
| 02 · Matemáticas | ✅ completo (6 notas) |
| 03 · Cinemática 1D | ✅ completo (10 notas) |
| 04 · Cinemática 2D | 🔄 en progreso |
| 05 · Fuerzas 1D | ⬜ en preparación |
| 06 · Fuerzas 2D | ⬜ pendiente |

Traducción, adaptación pedagógica e integración con actividades de campo por Luciano Lamaita, con autorización del autor original (licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)).

## Guías en PDF (formato paper)

Cualquier nota del portal puede convertirse en un PDF con formato de paper académico (dos columnas, tipografía Georgia, estilo APA), con las fórmulas KaTeX ya renderizadas. Los PDF viven en `static/files/pdf/` y se linkean desde la propia nota.

Piezas del mecanismo:

- **`assets/css/extended/print-paper.css`**: CSS que solo se activa en `@media print`. Define página A4 con márgenes 20mm×14mm, Georgia 9,8pt, dos columnas de texto, tablas en formato APA (3 líneas horizontales, sin grilla vertical), encabezados h2/h3 con filetes finos y pie de página con número de página. No afecta la vista en pantalla — de eso se ocupa `custom.css`.
- **`scripts/generate-pdf.mjs`**: script Node.js con Playwright (Chromium headless). Busca los `.md` con `pdf = true` en el frontmatter, levanta un servidor HTTP local sobre `./public/` (Hugo genera rutas absolutas de CSS/JS que no resuelven por `file://`) y, para cada nota, abre la página, activa `@media print`, espera a que KaTeX termine de renderizar, inyecta el byline (`Luciano Lamaita · profe.lemeit.ar · fecha`), expande a `column-span: all` las tablas que desbordan el ancho de columna, y llama a `page.pdf()` (A4, con número/total de página) guardando el resultado en `static/files/pdf/<slug>.pdf`.
- **`layouts/partials/extend_head.html`**: carga KaTeX desde CDN solo si la nota tiene `math = true`. Es el mismo KaTeX que se ve en pantalla el que Playwright renderiza antes de imprimir, así que las fórmulas del PDF salen idénticas a las de la web.
- **`resource-box`** en el `.md`: bloque HTML con clase `no-print` (no aparece en el PDF) con el link de descarga:

```html
<div class="resource-box no-print">

Recursos

📄 <a href="/files/pdf/nombre-nota.pdf">Descargar en PDF (formato paper)</a>

</div>
```

### Agregar el PDF a una nota nueva

**1. Frontmatter**

```toml
+++
title = 'Título de la nota'
math = true
pdf = true
+++
```

**2. `resource-box` al inicio del contenido**

```html
<div class="resource-box no-print">

Recursos

📄 <a href="https://profe.lemeit.ar/files/pdf/nombre-nota.pdf" target="_blank" rel="noopener">Descargar en PDF (formato paper)</a>

</div>
```

El slug del PDF es el path de la nota relativo a `content/`, sin `.md`. Por ejemplo, `content/notes/notas-fisica/cap03-cinematica-1d/mruv.md` genera `static/files/pdf/notes/notas-fisica/cap03-cinematica-1d/mruv.pdf`.

**3. Generar el PDF**

```powershell
cd C:\GitHub\aboutme
hugo                          # genera ./public/
node scripts/generate-pdf.mjs # genera los PDFs en static/files/pdf/
```

Requiere Node.js y Playwright instalados:

```powershell
npm install playwright
npx playwright install chromium
```

**4. Commitear todo junto**

```powershell
git add content/notes/ruta/nota.md
git add static/files/pdf/ruta/nota.pdf
git commit -m "nota: agrega PDF de <título>"
git push
```

### Fórmulas que no entran en una columna

Si una fórmula es más ancha que la columna de impresión (~83mm), no se achica automáticamente — hay que partirla en el `.md` usando `\begin{aligned}...\end{aligned}`:

```latex
$$
\begin{aligned}
  w_{real} &= \frac{w_{ideal}}{\eta} \\
            &= \frac{0{,}100}{0{,}72} = 0{,}139\ \tfrac{\text{kJ}}{\text{kg}}
\end{aligned}
$$
```

Las tablas anchas sí se expanden automáticamente a las dos columnas — el script detecta overflow y aplica `column-span: all`.

## El mapa de conceptos (`/conceptos`)

Un sitio Quartz v5 aparte, con notas en formato Obsidian, organizadas en 7 áreas temáticas: Mecánica, Gravitación, Materia, Termodinámica, Ondas, Electricidad y Magnetismo, Física Moderna. El `index.md` raíz tiene wikilinks explícitos a las 7 áreas para que el grafo muestre la red de conexiones ya desde la home, y el Explorer lateral respeta ese mismo orden temático (no alfabético) vía un `sortFn` propio en `quartz.ts`. Grafo local con `depth: 2` (desde una nota se ven sus vecinos directos y los de sus vecinos); grafo global disponible con un toggle.

## Diseño — paleta "X-Wing Poe Dameron"

Colores cálidos (crema + naranja quemado) compartidos entre Hugo y Quartz, con dark mode sincronizado manualmente entre ambos generadores:

| Variable | Valor (modo claro) | Uso |
|---|---|---|
| `--primary` | `#9B3D00` | Links, headings |
| `--secondary` | `#5C2200` | Texto secundario, hover |
| `--theme` / `--entry` | `#FAF7F2` | Fondo crema cálido |

En dark mode, el selector crítico es `:root[data-theme="dark"]` (no `.dark`) para igualar la especificidad de PaperMod v8+; los links necesitan `color: #B84800 !important` para ganarle a los selectores propios del tema.

## Instituciones donde se dictan las materias

| Materia | Institución |
|---|---|
| Introducción a la Física — 4° año | Colegio Madre Teresa + Instituto Niño Jesús (INJ) |
| Física — 5° año | Instituto Niño Jesús (INJ) |
| Física Clásica y Moderna — 6° año | Instituto Niño Jesús (INJ) |
| Laboratorio de Industrias — 7° TQ | EEST N°1 "Gral. Savio" |

## Notas de implementación

- **Dos generadores, un repo**: mantener sincronizada la paleta a mano entre `assets/css/extended/custom.css` (Hugo) y `quartz/quartz.config.yaml` (Quartz) es la principal fuente de trabajo manual de este proyecto — no hay un design system compartido como `design.lemeit.ar` en la Red Ambiental, porque Hugo y Quartz no comparten runtime.
- **Orden determinístico de secciones**: cada `_index.md` de `notes/` tiene un `weight` único (1–6); sin eso, Hugo ordena las secciones de forma indeterminada.
- **`public/` no se commitea**: la carpeta de salida de Hugo está en `.gitignore` — la genera Cloudflare Pages en cada deploy, igual que `site/` en esta wiki.

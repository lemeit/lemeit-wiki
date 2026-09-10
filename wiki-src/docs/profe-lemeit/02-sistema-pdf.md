# Sistema de generación de PDFs — profe.lemeit.ar

Convierte notas del sitio Hugo en PDFs con formato de paper académico (dos columnas, Georgia, estilo APA), con fórmulas KaTeX renderizadas. Los PDFs viven en `static/files/pdf/` y se linkean desde la nota.

## Componentes del sistema

- **`assets/css/extended/print-paper.css`**: CSS que solo se activa en `@media print`. Define página A4 con márgenes 20mm×14mm, fuente Georgia 9,8pt, dos columnas de texto, tablas en formato APA (3 líneas horizontales, sin grilla vertical), encabezados h2/h3 con filetes finos, pie de página con número de página. No afecta la vista en pantalla — eso lo maneja `custom.css`.
- **`scripts/generate-pdf.mjs`**: script Node.js que usa Playwright (Chromium headless). Flujo:
    1. Busca todos los `.md` en `content/` que tengan `pdf = true` en el frontmatter.
    2. Levanta un servidor HTTP local sobre `./public/` (necesario porque Hugo genera rutas absolutas de CSS/JS que no resuelven por `file://`).
    3. Para cada nota: abre la página en Chromium, activa `@media print`, espera que KaTeX termine de renderizar, inyecta el byline `Luciano Lamaita · profe.lemeit.ar · fecha`, detecta tablas que desborden el ancho de columna y las pasa a `column-span: all` automáticamente.
    4. Llama a `page.pdf()` con formato A4 y pie de página con número/total de páginas.
    5. Guarda en `static/files/pdf/<slug>.pdf`.
- **`layouts/partials/extend_head.html`**: carga KaTeX desde CDN solo si la nota tiene `math = true` en el frontmatter. El mismo KaTeX que se ve en pantalla es el que Playwright renderiza antes de imprimir — así las fórmulas del PDF son idénticas a las de la web.
- **`resource-box` en el `.md`**: bloque HTML con clase `no-print` (no aparece en el PDF) que muestra los links de descarga:

```html
<div class="resource-box no-print">

Recursos

📄 <a href="/files/pdf/nombre-nota.pdf">Descargar en PDF (formato paper)</a>

</div>
```

## Cómo agregar el PDF a una nota nueva

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

## Fórmulas que no entran en una columna

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

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

Deploy: cada push a `main` dispara un build automático en Cloudflare Pages (~1-2 min) — sin pasos manuales, a diferencia de `ema-saladillo` (que necesita `wrangler pages deploy`).

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

# lemeit-wiki

[![sitio: wiki.lemeit.ar](https://img.shields.io/badge/sitio-wiki.lemeit.ar-009688?style=flat-square&labelColor=555555)](https://wiki.lemeit.ar/)
[![build: MkDocs](https://img.shields.io/github/actions/workflow/status/lemeit/lemeit-wiki/wiki-build.yml?branch=main&style=flat-square&label=build&labelColor=555555)](https://github.com/lemeit/lemeit-wiki/actions/workflows/wiki-build.yml)

Wiki madre de documentación técnica y bitácora de los proyectos de **lemeit.ar**. Organizada por proyecto, todo público — sin nada que ocultar en la parte de desarrollo.

## Wiki publicada

👉 **[wiki.lemeit.ar](https://wiki.lemeit.ar/)**

## Proyectos documentados hoy

| Sección | Proyecto | Sitio | Repo |
|---|---|---|---|
| Red Ambiental | 🌬️ Monitoreo Ambiental Escolar | [app.lemeit.ar/aq/](https://app.lemeit.ar/aq/) | [lemeit-aq](https://github.com/lemeit/lemeit-aq) |
| Red Ambiental | 🌡️ EMA Saladillo | [app.lemeit.ar/emas/](https://app.lemeit.ar/emas/) | [lemeit-emas](https://github.com/lemeit/lemeit-emas) |
| Red Ambiental | 💧 Calidad del Agua | [app.lemeit.ar/wq/](https://app.lemeit.ar/wq/) | [lemeit-wq](https://github.com/lemeit/lemeit-wq) |
| — | 🎓 Eureka AI | [tutor.lemeit.ar](https://tutor.lemeit.ar) | `lemeit/eureka` (privado) |
| — | 👨‍🏫 Profe Lamaita | [profe.lemeit.ar](https://profe.lemeit.ar) | [aboutme](https://github.com/lemeit/aboutme) |

Más proyectos se suman como secciones nuevas a medida que se documentan (ver "Cómo agregar un proyecto" más abajo).

## Cómo funciona este repo

Sigue el mismo patrón que ya usa la wiki de [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/`):

```
wiki-src/docs/**/*.md   →  mkdocs build  →  site/  →  Cloudflare Pages (wiki.lemeit.ar)
      (fuente,               (build MkDocs,     (contenido
   se edita manualmente)     wiki-build.yml)   generado, no editar manualmente)
```

- El contenido fuente vive en `wiki-src/docs/`, en Markdown, **una carpeta por proyecto**.
- `.github/workflows/wiki-build.yml` valida MkDocs en modo estricto, sin commitear salida generada.
- Cloudflare Pages, proyecto `lemeit-wiki`, compila con `pip install -r wiki-src/requirements.txt && cd wiki-src && mkdocs build --site-dir ../site` y publica `site/` desde la rama `main`.
- El dominio `wiki.lemeit.ar` está asociado al proyecto Pages. `site/` es salida de compilación; no editarla ni commitearla.

## Editar una página existente

1. Editar el `.md` correspondiente dentro de `wiki-src/docs/<proyecto>/`.
2. Commitear y pushear — el build y el deploy a Pages son automáticos.

## Agregar un proyecto nuevo

1. Crear `wiki-src/docs/<nombre-proyecto>/index.md` (y las páginas que hagan falta).
2. Sumar la sección al `nav:` de `wiki-src/mkdocs.yml`.
3. Agregar la fila correspondiente a la tabla de arriba y a la de `wiki-src/docs/index.md`.
4. Commitear y pushear.

## Previsualizar en local (opcional)

```bash
cd wiki-src
pip install -r requirements.txt
mkdocs serve
```

y abrir `http://localhost:8000`.

## Estructura

```
lemeit-wiki/
├── wiki-src/
│   ├── mkdocs.yml
│   ├── requirements.txt
│   └── docs/
│       ├── index.md                          # Portada de la wiki madre
│       ├── CNAME                             # wiki.lemeit.ar (dominio custom de Pages)
│       ├── red-ambiental/
│       │   ├── index.md                      # Resumen de los 3 portales
│       │   ├── 01-aire-saladillo.md
│       │   ├── 02-ema-saladillo.md
│       │   ├── 03-agua-saladillo.md
│       │   └── 99-bitacora.md                # Bitácora cronológica de los 3 proyectos
│       └── eureka-tutor/
│           └── index.md
├── site/                                     # Generado automáticamente — no editar manualmente
└── .github/workflows/wiki-build.yml
```

## Origen

Nace en agosto de 2026, primero como wiki exclusiva de la Red Ambiental (repo `ambiental-wiki`, hoy renombrado a `lemeit-wiki`) para unificar la documentación que hasta entonces solo existía, parcialmente, como 4 documentos Word sueltos dentro de `ema-saladillo/docs/`. Se reorganizó como wiki madre de todos los proyectos de lemeit.ar poco después, sumando Eureka AI como segunda sección.

## Portadas e identidad — 28/10/2026

Se incorporó la sección [Portal Lemeit](wiki-src/docs/portal-lemeit/index.md), con versión en inglés, rutas de las portadas, configuración de publicación, cuatro modelos de Eureka y favicon Encuentro. Interacción se conserva como alternativa en Eureka; la L personal no se usa para la marca Lemeit. Se actualizaron los resúmenes de Eureka, Profe y Red Ambiental para explicar su relación con la nueva entrada común.

## Logos de la wiki — 28/10/2026

El logo general y favicon usan **Encuentro** en `wiki-src/docs/assets/logos/lemeit.svg`. El portal **Profe Lamaita** se representa mediante la **L caligráfica personal de Luciano**, tomada del SVG canónico de `lemeit/aboutme`, en `assets/logos/profe-lamaita.svg`; `profe.svg` conserva la misma figura para compatibilidad. Inicio y ficha de Profe, en español e inglés, usan el nombre nuevo para renovar la imagen en caché. No usar la L personal para representar a la marca Lemeit ni la antigua L tipográfica para Profe.

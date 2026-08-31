# wiki-lemeit

Wiki madre de documentación técnica y bitácora de los proyectos de **lemeit.ar**. Organizada por proyecto, todo público — sin nada que ocultar en la parte de desarrollo.

## Wiki publicada

👉 **[wiki.lemeit.ar](https://wiki.lemeit.ar/)**

## Proyectos documentados hoy

| Sección | Proyecto | Sitio | Repo |
|---|---|---|---|
| Red Ambiental | 🌬️ Aire Saladillo | [aq.lemeit.ar](https://aq.lemeit.ar) | [purpleair-saladillo](https://github.com/lemeit/purpleair-saladillo) |
| Red Ambiental | 🌡️ EMA Saladillo | [emas.lemeit.ar](https://emas.lemeit.ar) | [ema-saladillo](https://github.com/lemeit/ema-saladillo) |
| Red Ambiental | 💧 Calidad del Agua | [wq.lemeit.ar](https://wq.lemeit.ar) | [agua-saladillo](https://github.com/lemeit/agua-saladillo) |
| — | 🎓 Eureka Tutor | [tutor.lemeit.ar](https://tutor.lemeit.ar) | `lemeit/eureka` (privado) |

Más proyectos se suman como secciones nuevas a medida que se documentan (ver "Cómo agregar un proyecto" más abajo).

## Cómo funciona este repo

Sigue el mismo patrón que ya usa la wiki de [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/`):

```
wiki-src/docs/**/*.md   →  mkdocs build  →  site/  →  GitHub Pages (wiki.lemeit.ar)
      (fuente,               (GitHub Action,     (contenido
    se edita a mano)        wiki-build.yml)     generado, no editar a mano)
```

- El contenido fuente vive en `wiki-src/docs/`, en Markdown, **una carpeta por proyecto**.
- Cada push a `wiki-src/**` dispara `.github/workflows/wiki-build.yml`, que corre `mkdocs build --site-dir ../site` y commitea el resultado en `site/` (con `[skip ci]` para no generar un loop).
- GitHub Pages sirve la carpeta `site/` de la rama `main`, con dominio propio `wiki.lemeit.ar` (archivo `wiki-src/docs/CNAME`, que `mkdocs` copia sin tocar a `site/CNAME` en cada build).

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
wiki-lemeit/
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
├── site/                                     # Generado automáticamente — no editar a mano
└── .github/workflows/wiki-build.yml
```

## Origen

Nace en agosto de 2026, primero como wiki exclusiva de la Red Ambiental (repo `ambiental-wiki`, hoy renombrado a `wiki-lemeit`) para unificar la documentación que hasta entonces solo existía, parcialmente, como 4 documentos Word sueltos dentro de `ema-saladillo/docs/`. Se reorganizó como wiki madre de todos los proyectos de lemeit.ar poco después, sumando Eureka Tutor como segunda sección.

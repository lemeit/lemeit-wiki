# ambiental-wiki

Documentación técnica y bitácora unificada de la **Red Ambiental Saladillo**: los tres portales hermanos de monitoreo ambiental de Saladillo, Buenos Aires.

| Portal | Dominio | Repo |
|---|---|---|
| 🌬️ Aire Saladillo | [aq.lemeit.ar](https://aq.lemeit.ar) | [purpleair-saladillo](https://github.com/lemeit/purpleair-saladillo) |
| 🌡️ EMA Saladillo | [emas.lemeit.ar](https://emas.lemeit.ar) | [ema-saladillo](https://github.com/lemeit/ema-saladillo) |
| 💧 Calidad del Agua | [wq.lemeit.ar](https://wq.lemeit.ar) | [agua-saladillo](https://github.com/lemeit/agua-saladillo) |

## Wiki publicada

👉 **[lemeit.github.io/ambiental-wiki](https://lemeit.github.io/ambiental-wiki/)**

## Cómo funciona este repo

Este repo sigue el mismo patrón que ya usa la wiki de [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/`):

```
wiki-src/docs/*.md   →  mkdocs build  →  site/  →  GitHub Pages
     (fuente,             (GitHub Action,    (contenido
   se edita a mano)     wiki-build.yml)      generado, no editar a mano)
```

- El contenido fuente vive en `wiki-src/docs/*.md`, escrito en Markdown.
- Cada push a `wiki-src/**` dispara `.github/workflows/wiki-build.yml`, que corre `mkdocs build --site-dir ../site` y commitea el resultado en `site/` (con `[skip ci]` para no generar un loop).
- GitHub Pages sirve directamente la carpeta `site/` de la rama `main`.

## Editar la wiki

1. Editar o agregar archivos `.md` dentro de `wiki-src/docs/`.
2. Si es una página nueva, agregarla también al `nav:` de `wiki-src/mkdocs.yml`.
3. Commitear y pushear — el build y el deploy a Pages son automáticos.

Para previsualizar en local (opcional, requiere Python):

```bash
cd wiki-src
pip install -r requirements.txt
mkdocs serve
```

y abrir `http://localhost:8000`.

## Estructura

```
ambiental-wiki/
├── wiki-src/
│   ├── mkdocs.yml
│   ├── requirements.txt
│   └── docs/
│       ├── index.md                 # Portada
│       ├── 01-aire-saladillo.md     # Portal de calidad del aire
│       ├── 02-ema-saladillo.md      # Portal meteorológico (4 estaciones)
│       ├── 03-agua-saladillo.md     # Portal de calidad del agua
│       └── 99-Bitacora.md           # Bitácora cronológica de los 3 proyectos
├── site/                            # Generado automáticamente — no editar a mano
└── .github/workflows/wiki-build.yml
```

## Origen

Nace en agosto de 2026 para unificar la documentación técnica y la bitácora de desarrollo que hasta entonces solo existía, parcialmente, como 4 documentos Word sueltos dentro de `ema-saladillo/docs/` (`bitacora_ema_saladillo_v1..v4.docx`) — migrados a `99-Bitacora.md`. Ver esa página para el historial completo de los tres portales.

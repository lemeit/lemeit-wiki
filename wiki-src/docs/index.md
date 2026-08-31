# Red Ambiental Saladillo

Documentación técnica y bitácora de desarrollo de la red de monitoreo ambiental de Saladillo, Buenos Aires: tres portales hermanos que comparten la misma infraestructura de Cloudflare (Pages + Workers + D1) y el mismo sistema de diseño ([design.lemeit.ar](https://design.lemeit.ar)).

| Portal | Dominio | Qué mide | Repo |
|---|---|---|---|
| 🌬️ Aire Saladillo | [aq.lemeit.ar](https://aq.lemeit.ar) | Calidad del aire (PM1.0/PM2.5/PM10, VOC, CO2, NOx) — sensores PurpleAir y AirGradient en escuelas y jardines | [purpleair-saladillo](https://github.com/lemeit/purpleair-saladillo) |
| 🌡️ EMA Saladillo | [emas.lemeit.ar](https://emas.lemeit.ar) | Meteorología — temperatura, humedad, presión, viento, lluvia de 4 estaciones automáticas | [ema-saladillo](https://github.com/lemeit/ema-saladillo) |
| 💧 Calidad del Agua | [wq.lemeit.ar](https://wq.lemeit.ar) | Arsénico, nitratos, fluoruro, metales pesados y bacteriología de la red municipal | [agua-saladillo](https://github.com/lemeit/agua-saladillo) |

## Origen del proyecto

Nace en marzo de 2026 como proyecto educativo del espacio curricular Laboratorio de Industrias, 7° Año Técnico Químico, EEST N°1 "Gral. Savio" (Saladillo, Buenos Aires), a cargo del Ing. Luciano Lamaita. El punto de partida fue acceder programáticamente a los datos de la Estación Meteorológica Automática (EMA) del propio establecimiento — de ahí surgió EMA Saladillo, que luego se amplió a una red de 4 estaciones. En agosto de 2026 los tres proyectos (EMA, aire, agua) se armonizaron sobre una misma arquitectura de Cloudflare para poder integrarse entre sí a futuro. Ver la [Bitácora del proyecto](99-Bitacora.md) para el historial completo.

## Arquitectura compartida

Los tres portales siguen el mismo patrón:

- **Ingesta**: scrapers en Python (GitHub Actions o Cron Trigger de Cloudflare) que escriben en una base **Cloudflare D1** propia por proyecto.
- **API**: un **Cloudflare Worker** por proyecto expone esa base como API REST — pública, de solo lectura, sin autenticación, con CORS abierto (además de ingesta/administración protegida donde corresponde). Ver la página de cada portal para el detalle de endpoints.
- **Dashboard**: un `index.html` estático (HTML/CSS/JS vanilla, sin build ni framework) que consulta el Worker vía `fetch()`, publicado en **Cloudflare Pages**.
- **Diseño**: paleta, tipografía (JetBrains Mono) y componentes compartidos (header, selector de portales, footer versionado) vía [design.lemeit.ar](https://design.lemeit.ar) (`lemeit-theme.css` + `lemeit-common.js`).
- **Mapas**: tiles de CARTO Basemaps servidos vía proxy del propio Worker, para no exponer la API key en el HTML público.

## Sobre esta wiki

Generada con [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/), mismo stack que usa la wiki de [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/` → build automático vía GitHub Actions → `site/`). El código fuente de cada página vive en `wiki-src/docs/` de este repo.

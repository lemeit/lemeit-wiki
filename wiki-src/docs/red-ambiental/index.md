# Red de Monitoreo Ambiental

Documentación técnica y bitácora de desarrollo de la red de monitoreo ambiental de Saladillo, Buenos Aires: tres portales hermanos que comparten la misma infraestructura de Cloudflare (Pages + Workers + D1) y el mismo sistema de diseño ([design.lemeit.ar](https://design.lemeit.ar)).

Desde setiembre 2026 los tres viven bajo un solo dominio, **`app.lemeit.ar`**, con un prefijo de ruta por portal — un Worker gateway (repo [`gateway`](https://github.com/lemeit/gateway)) reverse-proxea cada uno hacia el Cloudflare Pages que lo sigue publicando sin cambios. Los dominios viejos (`aq`/`emas`/`wq.lemeit.ar`) siguen funcionando, redirigen solos al dominio nuevo.

| Portal | Dominio | Qué mide | Repo |
|---|---|---|---|
| ![AQ](../assets/logos/aq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } Monitoreo Ambiental Escolar | [app.lemeit.ar/aq](https://app.lemeit.ar/aq/) | Calidad del aire (PM1.0/PM2.5/PM10, VOC, CO2, NOx) — sensores PurpleAir y AirGradient en instituciones educativas de la Provincia de Buenos Aires | [lemeit-aq](https://github.com/lemeit/lemeit-aq) |
| ![EMA](../assets/logos/ema.svg){: width="22" style="vertical-align:middle;margin-right:6px" } EMAS | [app.lemeit.ar/emas](https://app.lemeit.ar/emas/) | Meteorología — temperatura, humedad, presión, viento, lluvia de estaciones automáticas en Saladillo y 25 de Mayo | [lemeit-emas](https://github.com/lemeit/lemeit-emas) |
| ![WQ](../assets/logos/wq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } Calidad del Agua | [app.lemeit.ar/wq](https://app.lemeit.ar/wq/) | Arsénico, nitratos, fluoruro, metales pesados y bacteriología de la red municipal | [lemeit-wq](https://github.com/lemeit/lemeit-wq) |

## Origen del proyecto

Nace en marzo de 2026 como proyecto educativo del espacio curricular Laboratorio de Industrias, 7° Año Técnico Químico, EEST N°1 "Gral. Savio" (Saladillo, Buenos Aires), a cargo del Ing. Luciano Lamaita. El punto de partida fue acceder programáticamente a los datos de la Estación Meteorológica Automática (EMA) del propio establecimiento — de ahí surgió EMA Saladillo, que luego se amplió a una red de 4 estaciones. En agosto de 2026 los tres proyectos (EMA, aire, agua) se armonizaron sobre una misma arquitectura de Cloudflare para poder integrarse entre sí a futuro. Ver la [Bitácora del proyecto](99-bitacora.md) para el historial completo.

## Arquitectura compartida

Los tres portales siguen el mismo patrón:

- **Ingesta**: scrapers en Python (GitHub Actions o Cron Trigger de Cloudflare) que escriben en una base **Cloudflare D1** propia por proyecto.
- **API**: un **Cloudflare Worker** por proyecto expone esa base como API REST — pública, de solo lectura, sin autenticación, con CORS abierto (además de ingesta/administración protegida donde corresponde). Ver la página de cada portal para el detalle de endpoints y ejemplos de uso.
- **Dashboard**: un `index.html` estático (HTML/CSS/JS vanilla, sin build ni framework) que consulta el Worker vía `fetch()`, publicado en **Cloudflare Pages**.
- **Diseño**: paleta, tipografía (JetBrains Mono) y componentes compartidos (header, selector de portales, footer versionado) vía [design.lemeit.ar](https://design.lemeit.ar) (`lemeit-theme.css` + `lemeit-common.js`).
- **Mapas**: tiles de CARTO Basemaps servidos vía proxy del propio Worker, para no exponer la API key en el HTML público.

Ver la [Bitácora del proyecto](99-bitacora.md) para el historial sesión por sesión, incluyendo el análisis microclimático (efecto isla de calor urbano en EMA-CS) hecho con los primeros datos de las 4 estaciones.

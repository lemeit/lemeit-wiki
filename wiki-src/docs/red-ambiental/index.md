# Red de Monitoreo Ambiental

Documentación técnica y bitácora de una red de monitoreo ambiental en la provincia de Buenos Aires, enmarcada en proyectos educativos de Educación Ambiental Integral. Los tres portales comparten infraestructura de Cloudflare y un sistema de diseño común.

Desde setiembre 2026 los tres viven bajo un solo dominio, **`app.lemeit.ar`**, con un prefijo de ruta por portal — un Worker gateway (repo [`gateway`](https://github.com/lemeit/gateway)) reverse-proxea cada uno hacia el Cloudflare Pages que lo sigue publicando sin cambios. Los dominios viejos (`aq`/`emas`/`wq.lemeit.ar`) siguen funcionando, redirigen solos al dominio nuevo.

| Portal | Dominio | Qué mide | Repo |
|---|---|---|---|
| ![AQ](../assets/logos/aq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } Aire Escolar | [app.lemeit.ar/aq](https://app.lemeit.ar/aq/) | Calidad del aire (PM1.0/PM2.5/PM10, VOC, CO2, NOx) — sensores PurpleAir y AirGradient en instituciones educativas de la Provincia de Buenos Aires | [lemeit-aq](https://github.com/lemeit/lemeit-aq) |
| ![EMA](../assets/logos/ema.svg){: width="22" style="vertical-align:middle;margin-right:6px" } EMAS | [app.lemeit.ar/emas](https://app.lemeit.ar/emas/) | Meteorología — temperatura, humedad, presión, viento, lluvia de estaciones automáticas en Saladillo y 25 de Mayo | [lemeit-emas](https://github.com/lemeit/lemeit-emas) |
| ![WQ](../assets/logos/wq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } Calidad del Agua | [app.lemeit.ar/wq](https://app.lemeit.ar/wq/) | Arsénico, nitratos, fluoruro, metales pesados y bacteriología de la red municipal | [lemeit-wq](https://github.com/lemeit/lemeit-wq) |

## Origen del proyecto

La iniciativa recupera experiencias de ciencia ciudadana y monitoreo ambiental iniciadas en 2023.

En 2026, los proyectos educativos de monitoreo evolucionaron hacia una infraestructura tecnológica común para meteorología, aire y agua. Ver la [bitácora del proyecto](99-bitacora.md).

## Arquitectura compartida

Los tres portales siguen el mismo patrón:

- **Ingesta**: scrapers en Python (GitHub Actions o Cron Trigger de Cloudflare) que escriben en una base **Cloudflare D1** propia por proyecto.
- **API**: un **Cloudflare Worker** por proyecto expone esa base como API REST — pública, de solo lectura, sin autenticación, con CORS abierto (además de ingesta/administración protegida donde corresponde). Ver la página de cada portal para el detalle de endpoints y ejemplos de uso.
- **Dashboard**: un `index.html` estático (HTML/CSS/JS vanilla, sin build ni framework) que consulta el Worker vía `fetch()`, publicado en **Cloudflare Pages**.
- **Diseño**: paleta, tipografía (JetBrains Mono) y componentes compartidos (header, selector de portales, footer versionado) vía [design.lemeit.ar](https://design.lemeit.ar) (`lemeit-theme.css` + `lemeit-common.js`).
- **Mapas**: tiles de CARTO Basemaps servidos vía proxy del propio Worker, para no exponer la API key en el HTML público.

Ver la [Bitácora del proyecto](99-bitacora.md) para el historial sesión por sesión, incluyendo el análisis microclimático (efecto isla de calor urbano en EMA-CS) hecho con los primeros datos de las 4 estaciones.


## Acceso desde Lemeit

La nueva [portada Lemeit](https://lemeit.ar/) ofrece accesos a Aire Escolar, meteorología y calidad del agua. Es un cambio de navegación; no se modificaron sensores, ingesta, bases ni APIs. [Registro de portadas e identidad](../portal-lemeit/index.md).

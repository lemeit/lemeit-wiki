# Environmental Monitoring Network

Technical documentation and development log for Saladillo's environmental monitoring network, Buenos Aires: three sibling portals that share the same Cloudflare infrastructure (Pages + Workers + D1) and the same design system ([design.lemeit.ar](https://design.lemeit.ar)).

Since September 2026 all three live under a single domain, **`app.lemeit.ar`**, with a path prefix per portal — a gateway Worker (repo [`gateway`](https://github.com/lemeit/gateway)) reverse-proxies each one to the Cloudflare Pages project that still publishes it, unchanged. The old domains (`aq`/`emas`/`wq.lemeit.ar`) keep working, they just redirect to the new domain on their own.

| Portal | Domain | What it measures | Repo |
|---|---|---|---|
| ![AQ](../assets/logos/aq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } School Air Quality Monitoring | [app.lemeit.ar/aq](https://app.lemeit.ar/aq/) | Air quality (PM1.0/PM2.5/PM10, VOC, CO2, NOx) — PurpleAir and AirGradient sensors in educational institutions across Buenos Aires Province | [lemeit-aq](https://github.com/lemeit/lemeit-aq) |
| ![EMA](../assets/logos/ema.svg){: width="22" style="vertical-align:middle;margin-right:6px" } EMAS | [app.lemeit.ar/emas](https://app.lemeit.ar/emas/) | Weather — temperature, humidity, pressure, wind, rain from automatic stations in Saladillo and 25 de Mayo | [lemeit-emas](https://github.com/lemeit/lemeit-emas) |
| ![WQ](../assets/logos/wq.svg){: width="22" style="vertical-align:middle;margin-right:6px" } Water Quality | [app.lemeit.ar/wq](https://app.lemeit.ar/wq/) | Arsenic, nitrates, fluoride, heavy metals and bacteriology from the municipal water network | [lemeit-wq](https://github.com/lemeit/lemeit-wq) |

## Project origin

It all starts in 2023, when Luciano Lamaita was selected as a Community Ambassador for the OpenAQ Community Ambassadors program. That's where the first low-cost sensors installed in Saladillo came from (AirGradient, Clarity Node-S, Atmotube), along with the citizen-science project "Saladillo Schools in Action for Clean Air," presented at school science fairs and technology expos.

Integrating that work into a dedicated portal came later, in March 2026, as an educational project of the Industrial Laboratory course, 7th-year Chemical Technician track, EEST N°1 "Gral. Savio" (Saladillo, Buenos Aires), led by Eng. Luciano Lamaita. The starting point was getting programmatic access to the school's own Automatic Weather Station (EMA) data — that's how EMA Saladillo came about, later expanded into a network of 4 stations. In August 2026 the three projects (weather, air, water) were harmonized onto the same Cloudflare architecture so they could eventually integrate with each other. See the [project log](99-bitacora.md) (Spanish only) for the full history.

## Shared architecture

All three portals follow the same pattern:

- **Ingestion**: Python scrapers (GitHub Actions or a Cloudflare Cron Trigger) that write to a project-specific **Cloudflare D1** database.
- **API**: a **Cloudflare Worker** per project exposes that database as a REST API — public, read-only, no authentication, open CORS (plus protected ingestion/admin endpoints where relevant). See each portal's page for endpoint details and usage examples.
- **Dashboard**: a static `index.html` (vanilla HTML/CSS/JS, no build step or framework) that queries the Worker via `fetch()`, published on **Cloudflare Pages**.
- **Design**: shared palette, typography (JetBrains Mono) and components (header, portal switcher, versioned footer) via [design.lemeit.ar](https://design.lemeit.ar) (`lemeit-theme.css` + `lemeit-common.js`).
- **Maps**: CARTO Basemaps tiles served through the Worker's own proxy, so the API key is never exposed in the public HTML.

See the [project log](99-bitacora.md) (Spanish only) for the session-by-session history, including the microclimate analysis (urban heat island effect at EMA-CS) done with the first 4 stations' data.

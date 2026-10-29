# ![AQ](../assets/logos/aq.svg){: width="36" style="vertical-align:middle;margin-right:8px" } School Air Quality Monitoring — app.lemeit.ar/aq

Air quality sensor network (PM1.0/PM2.5/PM10, VOC, CO2, NOx, temperature, humidity, pressure) installed in educational institutions and private homes across Buenos Aires Province — a pilot project in the Saladillo and 25 de Mayo districts. Combines **PurpleAir** and **AirGradient** sensors on the same database and the same dashboard.

Repository: [github.com/lemeit/lemeit-aq](https://github.com/lemeit/lemeit-aq)

## Architecture

```
PurpleAir API + AirGradient API
        ↓ (ingestion every 15 min)
cron-job.org (external, only active path) — GitHub Actions and Cloudflare Cron Trigger stay on standby
        ↓
Cloudflare D1 — "sensores" (metadata) and "lecturas" (time series) tables
        ↓
Worker "purpleair-saladillo-api" — own REST API (JSON/CSV)
        ↓
index.html (Cloudflare Pages) — static dashboard
```

Until August 2026 three ingestion triggers ran in parallel (Cloudflare Cron Trigger, GitHub Actions and cron-job.org) as deliberate redundancy against a Cloudflare bug that leaves the Cron Trigger registered but never firing. The `INSERT OR IGNORE` + `UNIQUE(sensor_index, timestamp)` on `lecturas` prevents duplicate rows, but doesn't prevent each active path from spending its own call to the PurpleAir API — with 4 new sensors about to be added, in September 2026 GitHub Actions was paused (`workflow_dispatch` stays available to run it manually) and **cron-job.org became the only active trigger**, to avoid paying double in API quota points. Full detail in the [repo README](https://github.com/lemeit/lemeit-aq#readme).

## Public API

No authentication, open CORS, meant for anyone to consume directly — not just the dashboard itself. There's also interactive documentation with examples at [app.lemeit.ar/aq/?tab=api](https://app.lemeit.ar/aq/?tab=api).

| Endpoint | Description |
|---|---|
| `GET /api/sensores` | Metadata for every active sensor |
| `GET /api/ultimas` | Latest reading for each sensor |
| `GET /api/historico/:sensor_index` | A sensor's history — `range=24h\|7d\|30d`, or `desde`/`hasta` (absolute date range, UTC), `limit` (max. 20000) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Map tile proxy to CARTO Basemaps |

The first three accept `&formato=csv` to download CSV instead of JSON.

## API usage guide

Base URL: `https://api.lemeit.ar/aq` (public domain shared with the other environmental network projects, via a Cloudflare Route — see "Architecture" above; the original `*.workers.dev` still works the same as a direct fallback, without the `/aq` prefix). `app.lemeit.ar/aq` serves the static dashboard, not the API (the old `aq.lemeit.ar` only redirects there); the dashboard itself (`index.html`, which includes the API documentation) uses this same API base. Every example below works pasted straight into a browser's address bar or with `curl`.

**Metadata for the active sensors:**

```bash
curl "https://api.lemeit.ar/aq/api/sensores"
```

**Latest reading for each sensor, as CSV:**

```bash
curl "https://api.lemeit.ar/aq/api/ultimas?formato=csv" -o ultimas.csv
```

**History for a single sensor, last 7 days:**

```bash
curl "https://api.lemeit.ar/aq/api/historico/12345?range=7d"
```

**Same sensor, absolute date range (UTC) and CSV:**

```bash
curl "https://api.lemeit.ar/aq/api/historico/12345?desde=2026-08-01&hasta=2026-08-31&formato=csv" -o agosto.csv
```

`desde`/`hasta` accept `YYYY-MM-DD` (from the start/end of that day in UTC) or `YYYY-MM-DD HH:MM:SS` for hour-level precision. When present, they replace `range`.

**From JavaScript (for example, for your own dashboard):**

```javascript
const resp = await fetch("https://api.lemeit.ar/aq/api/ultimas");
const sensores = await resp.json();
```

**From Python + pandas, straight into a DataFrame:**

```python
import pandas as pd
df = pd.read_csv("https://api.lemeit.ar/aq/api/historico/12345?range=30d&formato=csv")
```

Each sensor's `sensor_index` comes from `GET /api/sensores` — no need to guess it.

### Downloading a sensor's entire history

`GET /api/historico/:sensor_index` returns at most 20,000 rows per request (3,000 by default if `limit` isn't sent). A sensor with several months of data can easily go over that cap — in that case there's no way to fetch it all in a single request, it has to be split into date chunks with `desde`/`hasta` (one per month, for example) and the resulting CSVs merged:

```bash
curl "https://api.lemeit.ar/aq/api/historico/12345?desde=2026-01-01&hasta=2026-01-31&formato=csv" -o 2026-01.csv
curl "https://api.lemeit.ar/aq/api/historico/12345?desde=2026-02-01&hasta=2026-02-28&formato=csv" -o 2026-02.csv
# ...
```

The dashboard's own "Download CSV" button on the History tab has this same limitation (it downloads whatever is filtered on screen, with the same cap). If you need a sensor's complete history in one go — for an analysis, or as your own backup — write to [info@lemeit.ar](mailto:info@lemeit.ar) and you'll be sent a direct export from the database.

## Admin scripts

Two PowerShell scripts under the repo's `scripts/`, for quick queries from the console without opening a browser (the endpoints they use require a header, which can't be pasted directly into the address bar).

**View site visits** (`scripts/ver-visitas.ps1`) — queries the Worker's own admin endpoints (`/api/admin/visitas`, `/api/admin/resumen`), protected by the `ADMIN_KEY` secret:

```powershell
cd scripts
.\ver-visitas.ps1                    # summary: 24h, 7d, top routes, top countries
.\ver-visitas.ps1 -Modo visitas      # last 200 visits, one by one
.\ver-visitas.ps1 -Modo visitas -Limit 50
```

Prompts for `ADMIN_KEY` on the console (hidden while typing), or it can be set for the whole PowerShell session with `$env:PA_ADMIN_KEY = "your_key"` so it isn't asked again.

**View PurpleAir API usage** (`scripts/ver-uso-purpleair.ps1`) — queries `api.purpleair.com` directly (doesn't go through our Worker) for how many points are left and at what rate they're being spent:

```powershell
cd scripts
.\ver-uso-purpleair.ps1
```

Prompts for `PURPLEAIR_API_KEY` (the same one set with `wrangler secret put`), or it can be set with `$env:PA_API_KEY = "your_key"` to avoid re-typing it. This particular query **doesn't spend quota points** — PurpleAir explicitly marks it as "a free API call, so query it as you need" (confirmed by their staff on the official API forum, April 2026), so it can be run as many times as needed.

## Sensors and AE codes

Each PurpleAir sensor in the network has its own **project code** (`AE-NN`), independent of the `sensor_index` assigned by PurpleAir.

| Code | `sensor_index` | Institution | District / setting | Address | CUE | PurpleAir registration |
|---|---|---|---|---|---|---|
| AE-00 | 198617 | Colegio Secundario Madre Teresa | Saladillo / rural | Calle Jauretche e Yrigoyen | — | 2023-11-28 (initial); 2024-12-03 (school registration) |
| AE-01 | 298687 | Esc. de Educación Artística N°1 "Lola Mora" | 25 de Mayo / urban | Calle 7 between 23 and 24 | 061617500 | 2026-10-05 |
| AE-02 | 298701 | Esc. de Educación Secundaria N°2 "Manuel Pardal" | Saladillo / urban | Av. Pereyra 3046 | 060342300 | 2026-10-05 |
| AE-03 | 298691 | Instituto Niño Jesús | — | — | — | 2026-10-28 |
| AE-04 | — | Del Valle (reservado) | — | — | — | Pendiente |

AE-01, AE-02 and AE-03 are PurpleAir PA-II-FLEX units. AE-03 is assigned to Instituto Niño Jesús (298691); AE-04 is reserved for Del Valle, with no sensor_index assigned. The convention was defined for the PurpleAir sensors; the AirGradient ones (synthetic ID 900000+) don't have a code assigned yet.

### PurpleAir registration and physical installation are different dates

Registration confirmations supplied by the owner on 2026-10-29 establish **AE-01 and AE-02: 2026-10-05; AE-03: 2026-10-28**. The previous 2026-10-06 registration date for AE-01/AE-02 is corrected; it is not reinterpreted as an installation date. Physical installation is recorded separately.

| Code | Device ID | Latitude declared at registration | Longitude declared at registration |
| --- | --- | --- | --- |
| AE-01 | `ec:64:c9:cb:9f:3` | -35.43677574362913 | -60.168800362812966 |
| AE-02 | `f0:24:f9:c2:cf:2d` | -35.63495676824009 | -59.7735131464144 |
| AE-03 | `fc:f5:c4:ae:ca:d6` | -35.640825 | -59.778565 |

The emails record outdoor location, public visibility and the default sending interval (~90 s), separate from LEMEIT ingestion. Online status refers only to when the email was sent, not current connectivity or permanent installation. Mini guides put registration date and coordinates under **Observations**; the installation date is left for the actual mounting event. Read keys and complete emails are excluded. See the [versioned registration record](https://github.com/lemeit/lemeit-aq/blob/main/docs/instalacion/REGISTRO_PURPLEAIR.md).

AE-00 is the same device, ID `c8:c9:a3:f:93:af`: initially configured as `saladillo_home` on **2023-11-28**, then registered as `Madre Teresa High School` on **2024-12-03** after more than a year at the owner's home, according to the owner. The school confirmation gives sensor index **198617** and declared coordinates **-35.66630, -59.80716**. The initial email does not provide a sensor index; hardware identity connects the records. Neither registration date establishes the exact mounting date or a future relocation date. Home coordinates, contact details and read keys are excluded. The emails do not establish the specific hardware model.

### Code convention

- `AE-00` is the network's first sensor, from before the project; `AE-01` onwards are assigned in installation order.
- The code is **stable**: if a sensor moves to another school, the code stays and only the institution changes.
- In the D1 database (`sensores` table) it is stored in the `codigo` column (unique), and the installation date in `fecha_instalacion`. The API returns both in `/api/sensores` and `/api/ultimas`.
- On the portal, each card shows the institution as its title and the sensor name (`AE-NN`) below it.
- The Worker **does not overwrite** `nombre`, `institucion`, `codigo` or `fecha_instalacion` during ingestion; it only updates latitude and longitude from PurpleAir.

### How to add a sensor

1. Register it on PurpleAir with the name `AE-NN`, the school's coordinates, outdoor location and public visibility.
2. Note the `sensor_index` PurpleAir assigns (this is not the Device ID).
3. Update the Worker's `SENSOR_INDEXES` secret. **The secret is replaced entirely** (in Cloudflare: *Rotate*, then *Deploy*): paste the full list, with the sensors already active plus the new one, comma-separated with no spaces.
4. Wait for the next ingestion run (or trigger it manually): the row is created in `sensores`.
5. Fill in `codigo`, `fecha_instalacion`, `institucion` and `notas` (CUE and provincial code) with an `UPDATE` from the D1 console.
6. Check with a `SELECT` that the row is complete.

An `UPDATE` on a `sensor_index` that doesn't exist yet raises no error and changes nothing.

## Roadmap

> Update, 28 October 2026: AE-01 is still awaiting physical installation according to the latest owner report; AE-02 is operational. AE-03 is online and visible according to the owner, with a recent D1 reading verified independently. Its project code `AE-03` was added with explicit authorization and checked by SELECT. AirGradient 900002 and 900003 have old readings; current connectivity is unverified. Rotate created a Worker version that still required deployment in this incident. See the [Spanish log](99-bitacora.md) and [safe procedure and rollback](https://github.com/lemeit/lemeit-aq/blob/main/docs/OPERACION_PURPLEAIR.md). The hardware description below records an earlier state.


**Hardware status (October 2026)**: all 5 **PurpleAir** sensors are in the project's hands. The only one running until now was AE-00, at Colegio Secundario Madre Teresa, currently pending relocation. AE-01 (25 de Mayo) and AE-02 (Saladillo) are already registered on PurpleAir and on the portal, and their installation is planned for October 6; AE-03 and AE-04 don't have an assigned destination yet. The 4 new units were a donation from [PurpleAir Collective](https://community.purpleair.com/t/purpleair-collective-june-july-2024/8771) (mid-2024 call, based on a needs proposal submitted by the author — the project placed 2nd in that round). The 2 **AirGradient** sensors are also in hand and active, currently connected at a private home for testing, pending a decision on which institutions they'll be installed in.

**Phase 2 — confirmed expansion (institutional meeting on 9/16/2026, convened by the Regional Education Office)**: 5 schools — 3 in Saladillo and 2 in the **25 de Mayo** district — one urban and one rural site in each district, expanding the network beyond Saladillo for the first time. 4 confirmed; the fifth (Colegio Secundario Madre Teresa) is being relocated. Meeting documentation: deployment plan and specification/requirements sheet in the [repo's `docs/`](https://github.com/lemeit/lemeit-aq/tree/main/docs). See also the [EMA Saladillo roadmap](02-ema-saladillo.md) (Spanish only), which adds a station in 25 de Mayo for the same reason. This expansion opens the door to a future combined report (air + weather) with spatial analysis between sensors — not immediate today, since the EMA stations and the air sensors don't share a physical location yet.

## Technical milestones

- **Deduplication**: when a sensor loses connection, PurpleAir keeps returning its last frozen `last_seen` — without `UNIQUE(sensor_index, timestamp)`, each cron run generated a new row with the same timestamp, producing duplicate/triplicate cards on the dashboard.
- **AirGradient (August 2026)**: added as a second sensor provider, reusing the same tables (`proveedor` tells the two apart) instead of a separate portal.
- **Tile proxy**: CARTO started requiring an API key to serve tiles; instead of exposing it in the public HTML, the Worker acts as a proxy and adds the key server-side (`CARTO_API_KEY` as a secret).
- **Public API (August 2026)**: the read endpoints, which already existed to feed the dashboard itself, were documented and expanded (absolute date range, CSV export) so third parties — another school department, or the municipality, for example — can consume the data without depending on the dashboard.
- **Visit tracking (September 2026)**: new `visitas` table in D1, written in the background via `ctx.waitUntil` so it never delays the real response, plus two read-only endpoints (`/api/admin/visitas`, `/api/admin/resumen`) protected by an `ADMIN_KEY` secret sent as the `X-Admin-Key` header — same scheme already used by `X-Ingest-Key` — instead of `?key=...` in the URL, so the key doesn't end up stuck in browser history or in access logs/intermediate proxies. `scripts/ver-visitas.ps1` was added to query it from PowerShell, since there `curl` is an alias for `Invoke-WebRequest` with different header syntax than real `curl` (real `curl.exe`, or `-Headers @{...}`, is needed).
- **History — configurable report and PDF export (September 2026)**: added a custom date-range filter (`desde`/`hasta`), a parameter filter via checkboxes (including PM1.0/PM10, which PurpleAir and AirGradient already reported but weren't shown), and a PDF export with NOAA-style climatological report formatting — plain monospaced text (jsPDF's Courier font, no embedded font files), always on a white background, independent of the rest of the site's light/dark theme. jsPDF loads from cdnjs with an automatic fallback to jsDelivr if the primary CDN fails (seen in production: a content blocker prevented the script from loading and left `window.jspdf` undefined, unhandled — now there's a clear notice to the user instead of a broken page). The PDF, besides downloading, now also opens in a new tab (blob URL, the browser's native viewer) within the same click, so it can be viewed without having to go find the downloaded file.
    - The on-screen "Report" view (a Classic/Report toggle next to the normal table) turned out redundant after real use — the PDF already always looked white/report-style. The toggle was removed: the on-screen table always uses the site's normal format, and the report style stayed reserved for the PDF. Along the way, `accent-color` was added to the site's checkboxes (the browser's default blue clashed with the palette).
    - Terminology: "Estación" (Station) was replaced with "Sitio" (Site) — in the PDF header and other places naming a sensor's location — to avoid clashing with "weather station" from the sibling EMA project.
- **Chart — site comparison (September 2026)**: new mode ("Compare sites") where you pick one parameter — not several, to avoid mixing units on the same axis — and several sites at once, each drawn as a differently-colored line on the same Y axis. Since not every site measures at the exact same instant (each one polls on its own), the time axis is built by merging the timestamps of all selected sites, with `spanGaps` so lines aren't cut at each site's own gaps — a known limitation, to be improved in the future with rounded-interval bucketing.
- **Installable PWA (September 2026)**: the dashboard can be installed as an app (`manifest.json`, a "maskable" icon for Android, `sw.js`). When opened installed — or with `?pwa=1` in the URL, which is the manifest's `start_url` — it starts directly on the "Current" tab instead of "Map": the speedometer-style AQI gauges are the most useful landing view on a phone. Launch detection via `matchMedia("(display-mode: standalone)")` plus `navigator.standalone` (covers Safari/iOS, which doesn't support that media query well). The service worker only caches the "app shell" (HTML, manifest, icons); live data (the own API) and CDN libraries (Chart.js, jsPDF, Leaflet, fonts, design.lemeit.ar) always go straight to the network, so readings and library versions never go stale.
- **Chart — zoom by selection (September 2026)**: added `chartjs-plugin-zoom` (+ `hammerjs` for touch gestures) to the Chart tab, in both modes (a single site's history and "Compare sites"): mouse drag for box-zoom selection, scroll wheel to zoom in/out, touch pinch on mobile, and a "Reset zoom" button that only shows up once a zoom has been applied. Same dual-CDN fallback pattern already used for jsPDF (jsDelivr first, cdnjs if it fails).
    - **Fix**: on the first deploy, dragging to select ended up zooming in the opposite direction from the drag, worse the more nested zooms were applied (zooming into an already-zoomed view). Cause: panning (moving the view by dragging) and zoom-by-selection were bound to the same gesture — a plain drag, no modifier key — and fought each other; panning shifted the axis while the selection rectangle was still being drawn. Fixed by requiring Ctrl+drag to pan, leaving a plain drag exclusively for zoom selection. Along the way, the X axis was switched from a category scale (text labels) to a numeric (`linear`) scale, with times shown via a tick callback, because `chartjs-plugin-zoom` doesn't handle nested zooms well on a category axis.
- **Wiki links**: the shared footer (`lemeit-common.js`, used by all three portals) and each portal's "About" section now link to [wiki.lemeit.ar](https://wiki.lemeit.ar) — until now the wiki wasn't mentioned from any portal.
- **AQ → AE logo and single `app.lemeit.ar` domain (September 2026)**: the logo initials changed from "AQ" (Air Quality, inconsistent with the rest of the Spanish-language site) to **AE** (Aire Escolar, "School Air"), in its own teal color (`#009688`) instead of the site's general orange. Alongside this, the network's three portals moved to live under a single domain (`app.lemeit.ar/aq`, `/emas`, `/wq` — see the [Network architecture](index.md)); `aq.lemeit.ar` still works, it just redirects. `manifest.json` widened its `scope` from `/aq/` to the whole new domain, so switching portals from the installed app doesn't kick out to the browser.
- **API documentation and credits inside the portal**: `api.html` and `creditos.html` went from standalone pages to tabs of `index.html` (`?tab=api`, `?tab=creditos`). The original files remain only as redirects so external links don't break, including the ones from this wiki.
- **Own code per sensor (October 2026)**: new `codigo` column in `sensores` (migration 008) with the `AE-NN` convention, independent of PurpleAir's `sensor_index` and stable if a sensor moves to another school. The installation date uses the `fecha_instalacion` column, which already existed in the schema. See "Sensors and AE codes" above for the procedure to add a sensor.

See the [project log](99-bitacora.md) (Spanish only) for the full session-by-session detail.

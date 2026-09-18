# ![EMA](../assets/logos/ema.svg){: width="36" style="vertical-align:middle;margin-right:8px" } EMAS — app.lemeit.ar/emas

Weather network of automatic stations in Saladillo and 25 de Mayo: temperature, humidity, pressure, wind, rain and other parameters, comparable to each other on a common time reference.

Repository: [github.com/lemeit/lemeit-emas](https://github.com/lemeit/lemeit-emas)

## The stations

| Code | Name | Organization / owner | Equipment | Access method |
|---|---|---|---|---|
| EMA-EET | EEST N°1 "Gral. Savio" | SNIH / INA — RMET Network | Tecmes | JSON POST API (expired SSL certificate on the server → requires `verify=False`) |
| EMA-CFR | Centro de Formación Rural | CFR Saladillo | Davis Instruments | HTML scraping (BeautifulSoup) |
| EMA-DC | Civil Defense — Airfield | Municipio Saladillo | Davis / Meteobridge | OCR on a camera image (Tesseract + Pillow) — exposes no data endpoint at all |
| EMA-CS | Clima Saladillo — Falucho neighborhood | Private individual | Davis / Meteotemplate | Public AJAX endpoint (clean JSON) |
| EMA-25C | 25Clima — 25 de Mayo | N-TecLab / SS Desarrollos (third party) | Ecowitt-compatible (WU: `IDEMAY14`) | Public Weather Underground API (PWS) |

## Architecture

```
5 Python scrapers (GitHub Actions, hourly cron)
        ↓ (Cloudflare HTTP API)
Cloudflare D1 — unified "mediciones" table (column "estacion")
        ↓
"ema-saladillo-api" Worker — compatible with the PostgREST format Supabase used
        ↓
index.html (Cloudflare Pages) — static dashboard
```

Until August 2026 the database was Supabase (PostgreSQL), with 4 separate tables. The entire history (~30,200 rows) was migrated to a single unified D1 table; the compatibility Worker exposes the same routes and query style (`select=`, `order=`, `limit=`, `column=eq.value`) so the dashboard wouldn't need to be rewritten.

## How the scrapers work

Five Python scripts in [`scrapers/`](https://github.com/lemeit/lemeit-emas/tree/main/scrapers), one per station — `snih_saladillo_v3.py` (EET), `cfr_saladillo.py`, `dc_saladillo.py`, `cs_saladillo.py` and `wu_25demayo.py` (25C) — each with its own access method (see the table above) and, accordingly, its own kind of fragility: EMA-DC depends on the OCR reading an image correctly; EMA-CFR and EMA-CS depend on a third-party site's HTML/JSON not changing format without notice; EMA-25C depends on the Weather Underground API's availability.

All 5 share the same internal structure: a `ciclo()` function does all the work of one run (fetch the data from the source, save it to D1, print a console summary) and returns `True`/`False` depending on whether it succeeded. Command-line flags, common to all 5:

- **No flags** — the mode the GitHub Actions cron uses: runs `ciclo()` once and exits with code `1` if something went wrong (see "Automation" below — needed so the failure gets recorded).
- **`--loop N`** — continuous mode, runs every N seconds without stopping. Meant for manual/supervised use (live testing from a terminal); never used in GitHub Actions, and deliberately doesn't stop on a one-off failure (a continuous manual check closing itself would be odd).
- **`--nod1`** — doesn't write to D1, to test the scraping against the source without touching the database.
- **`--csv`** — also exports a local CSV.

## How the data is stored

Everything goes into a single `mediciones` table in Cloudflare D1 (SQLite), with an `estacion` column to tell the source apart (`EMA-EET` | `EMA-CFR` | `EMA-DC` | `EMA-CS` | `EMA-25C`). Full schema in [`d1/schema.sql`](https://github.com/lemeit/lemeit-emas/blob/main/d1/schema.sql): besides the reading's own columns (`codigo`, `parametro`, `unidad`, `valor`, `valor_texto`, `fecha_hora_utc`), a unique index on `(estacion, parametro, fecha_hora_utc)` makes `INSERT OR IGNORE` automatically drop any duplicate row — useful since the cron and a manual run can overlap, or a scraper can retry.

All 5 scrapers write through a shared helper, [`scrapers/d1_writer.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/d1_writer.py) — nothing inserts into D1 directly. Two details of that helper are worth knowing, because both came out of real problems:

- **Local-to-UTC time conversion**: accepts both 4-digit and 2-digit years in the source date (`DD/MM/YYYY` and `DD/MM/YY`). This isn't cosmetic — EMA-CFR's source changed its date format without notice at some point in September 2026, and for almost a month **every** one of its rows was silently dropped for not matching the expected format (see the log, September 18 session).
- **Safety net for "zero rows saved"**: if there are rows with a valid value to save but **none** end up inserted (say, the date format changes again and neither expected pattern matches anymore), `guardar_en_d1()` raises an error instead of silently returning `0`. That's exactly what happened to EMA-CFR: the scraper ran "fine" (no exception at all), hour after hour, while each row got dropped one by one inside the helper — invisible both to the scraper's exit code and to GitHub Actions. With this net, the same problem (or a similar one) now surfaces as a real, auditable failure.

## Automation — GitHub Actions

Workflow: [`.github/workflows/scrapers.yml`](https://github.com/lemeit/lemeit-emas/blob/main/.github/workflows/scrapers.yml).

- **Hourly cron** (`5 * * * *`): runs all 5 scrapers, one per step, in a single job.
- **`workflow_dispatch`**: lets the workflow be run by hand from GitHub's Actions tab ("Run workflow"), without waiting for the cron — useful for testing a fix without waiting for the next hour.
- **Daily cron** (`5 11 * * *`, 08:05 AR): a separate job, unrelated to EMAS — it runs [`scrapers/supabase_ping.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/supabase_ping.py), which pings a *different* Supabase project than the one EMAS used to use, shared with other personal apps of the author, so its free tier doesn't get paused for inactivity. It ended up in this workflow for historical convenience (this is the same repo where the rest of the scraping used to live), but it's independent of EMAS's migration to D1: it still serves its purpose as long as those other apps keep running on that shared Supabase project.

**An important detail for reading the results correctly**: every scraper step has `continue-on-error: true` in the workflow — on purpose, so that if one station fails the other 4 still get attempted (without it, a failure would stop the whole job at its first failing step). The consequence is that even when an individual scraper exits with a failure code, **the whole run still shows green in the main Actions list** — GitHub only marks that specific step with a warning triangle if you open that particular run's details. In other words: the exit code makes the problem *recorded* and *auditable*, but it isn't enough to notice at a glance from the Actions list.

That's why actually confirming the 5 stations are healthy takes an active check, not just looking at whether Actions is green:

- [`tools/audit_timestamps.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/audit_timestamps.py): a read-only audit script (doesn't touch anything in the repo or write to D1, only reads the public API) that compares, per station, the time of day when temperature hits its minimum and maximum against what's expected (early morning / afternoon) — a mismatch signals a timezone problem or a stored time that isn't the real one — and how long it's been since new data arrived (a "silent" station). Run by hand: `python tools/audit_timestamps.py` (or `--dias 14` for a shorter window than the 30-day default).
- Manually opening the relevant Actions run and expanding each step, if one station in particular is suspect.

## Public API

No authentication, open CORS. Interactive documentation with examples: [app.lemeit.ar/emas/api.html](https://app.lemeit.ar/emas/api.html).

| Endpoint | Description |
|---|---|
| `GET /rest/v1/mediciones_ema` \| `mediciones_cfr` \| `mediciones_dc` \| `mediciones_cs` \| `mediciones_25c` | Raw readings for one station — `select`, `order`, `limit`, `codigo=eq.X`, `parametro=eq.X`, `horas=N` (relative window), or `desde`/`hasta` (absolute UTC date range) |
| `GET /rest/v1/v_temperatura_comparativa` | Hourly average temperature for all 5 stations, in parallel columns |
| `GET /rest/v1/v_ema_armonizada` | One normalized parameter across the 5 stations (temperature, humidity, pressure, wind, rain, etc.) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Map tile proxy to CARTO Basemaps |

The first three accept `&formato=csv`.

`order` only looks at the `.asc`/`.desc` suffix — the Worker always sorts by `fecha_hora_utc` regardless of what column name is put before the dot (inherited from how it was built for Supabase/PostgREST). In practice `order=timestamp.desc` and `order=fecha_hora_utc.desc` do exactly the same thing; there's no way to sort by any other column.

## API usage guide

Base URL: `https://api.lemeit.ar/emas` (public domain shared with the other environmental-network projects, via a Cloudflare Route — same scheme as `api.lemeit.ar/aq`; the original `*.workers.dev` — `ema-saladillo-api...workers.dev` — still works the same as a direct fallback, without the `/emas` prefix). `app.lemeit.ar/emas` serves the static dashboard, not the API (the old `emas.lemeit.ar` just redirects there) — the repo's own `index.html` and `api.html` use this same API base. Routes follow the PostgREST style inherited from Supabase: filters like `column=eq.value`, ordering with `order=column.desc`, limit with `limit=N`.

**Last 100 temperature readings for one station:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100"
```

**Same query but as CSV, to open directly in a spreadsheet:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100&formato=csv" -o temp_eet.csv
```

**Absolute date range (UTC) instead of `horas`:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_cfr?parametro=eq.Lluvia&desde=2026-08-01&hasta=2026-08-31&formato=csv" -o lluvia_agosto.csv
```

**Compare temperature across all 5 stations in parallel, last 48 hours:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/v_temperatura_comparativa?horas=48"
```

**Any parameter harmonized across the 5 stations:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/v_ema_armonizada?parametro=eq.Humedad&horas=24"
```

**From Python + pandas:**

```python
import pandas as pd
df = pd.read_csv("https://api.lemeit.ar/emas/rest/v1/v_temperatura_comparativa?horas=720&formato=csv")
```

If a query returns an empty CSV, that's probably not an error: that station may simply have no data in the requested window (for example, a transmission outage). It's worth trying without `formato=csv` first, or with a wider window (`horas=720`), to confirm whether there's data before assuming something's wrong.

## EMA-25C — the fifth station, in 25 de Mayo

In August/September 2026 a fifth station was added, in the 25 de Mayo district — the first in the network outside Saladillo, in line with the expansion of [School Air Quality Monitoring](01-aire-saladillo.md#roadmap) into that same district. Unlike the other 4, **EMA-25C is not a station the project owns**: it's the public 25Clima station (`25clima.ar`), operated by a third party (N-TecLab / SS Desarrollos) and registered on the Weather Underground network as `IDEMAY14`. It's queried through the public Weather Underground API — no direct access from the operator is needed, anyone with their own WU API key can read someone else's public stations (see [`scrapers/wu_25demayo.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/wu_25demayo.py) in the repo). Being a third-party station, it's still being onboarded: the operator still needs to be contacted to confirm its permanent status on the public dashboard. Because of its distance from the rest of the network (~80 km), it also doesn't take part in the dashboard's spatial interpolation (heat map).

With this regional expansion, the project stopped being called "EMA Saladillo" and became **EMAS**. The underlying goal, just as with School Air Quality Monitoring, is to enable combined reports (EMA + AQ) with spatial analysis — currently limited because the EMA stations and the air sensors aren't co-located.

## Technical milestones

- **Civil Defense OCR**: the site exposes no data endpoint at all — everything is overlaid as text on a JPG image. Solved by extracting white pixels (instead of direct grayscale, which loses white text on colored backgrounds) + Tesseract, with two-layer validation (plausible physical range + time delta) before inserting.
- **Time harmonization**: the 4 stations transmit at different frequencies; the `v_ema_armonizada` view normalizes by hour using an `IMMUTABLE` function (`hora_ar()`), needed because `date_trunc` isn't `IMMUTABLE` in PostgreSQL.
- **Migration to GitHub Actions (March 2026)**: the original system depended on Windows Task Scheduler on a physical PC — if it turned off, data was lost. See the log for details of the migration and why the Windows tasks were failing in that context (`PATH` missing the user's Python install).
- **Migration to Cloudflare D1 (August 2026)**: part of harmonizing the three portals onto the same infrastructure. Of the 30,213 rows exported from Supabase, 9 were discarded for a corrupted timestamp (a historical OCR error).
- **Public API (August 2026)**: same approach as School Air Quality Monitoring — the existing routes were documented and given an absolute date range and CSV export.
- **Fifth station via Weather Underground (September 2026)**: adding EMA-25C required its own Weather Underground API key, which is only generated once the account has at least one "active" device (with recent real data) — without owning a physical station, this was solved by activating a placeholder device with real data from a PurpleAir sensor already in the School Air Quality Monitoring network (see [`tools/subir_a_wu.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/subir_a_wu.py)). See the log for the full details.
- **Own brand color and single `app.lemeit.ar` domain (September 2026)**: EMAS's favicon moves from a stray blue to a teal from the same tonal family as School Air Quality Monitoring's green (`#0097A7`). Along with this, the network's three portals move under a single domain (`app.lemeit.ar/aq`, `/emas`, `/wq` — see the [Network's architecture](index.md)); `emas.lemeit.ar` keeps working, it just redirects. The header's station selector, which on small screens covered the rest of the controls (connection status, portal switcher, theme), moves to its own row with horizontal scroll.
- **API under `api.lemeit.ar/emas` (September 2026)**: `emas.lemeit.ar/rest/v1/*` never actually served data — the domain pointed at the static Pages site, which can't reach the D1 database; the real API always lived only on the `*.workers.dev` domain (that's what `index.html` itself calls it, via `SUPA_URL`). A Cloudflare Route is added to the same Worker (`api.lemeit.ar/emas/*`) so it gets its own clean public URL, just like School Air Quality Monitoring already has at `api.lemeit.ar/aq`. The original `*.workers.dev` keeps working the same as a fallback.
- **Health audit of the 5 stations and the end of EMA-CFR's silent data loss (September 2026)**: it turned out all 5 scrapers could fail (a real exception, or saving 0 rows to D1) without any GitHub Actions run showing red — `continue-on-error: true` plus an exit code that was always 0 left the problem completely invisible. Fixed in two layers: each scraper now exits with a non-zero code if its cycle fails, and `scrapers/d1_writer.py` now raises an error if there are rows to save but none end up inserted (it used to silently return `0`). That second layer is what found the real problem: EMA-CFR had been losing 100% of its data for almost a month because the source changed its date format (from 4-digit to 2-digit year) without notice, and each row was silently dropped one by one inside the helper. [`tools/audit_timestamps.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/audit_timestamps.py) was also added as an active health check for the 5 stations (diurnal cycle + data freshness), because even with the exit codes fixed, an Actions run still doesn't show red at a glance (see "Automation" above). See the project log for the full details, including the investigation into the "Rosa de vientos" (wind rose) rendering bug that came out of the same session.

See the [project log](99-bitacora.md) (Spanish only) for the full history, including the microclimate analysis (urban heat island effect at EMA-CS) done with the first 4 stations' data.

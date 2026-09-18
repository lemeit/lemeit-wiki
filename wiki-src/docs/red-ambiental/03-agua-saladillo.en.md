# ![WQ](../assets/logos/wq.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Water Quality — app.lemeit.ar/wq

Monitoring of tap water quality in Saladillo: arsenic, nitrates, nitrites, fluoride, heavy metals and bacteriological parameters (total coliforms, *E. coli*, *Pseudomonas aeruginosa*) across dozens of points in the municipal network (pumps, schools, kindergartens, households).

Repository: [github.com/lemeit/lemeit-wq](https://github.com/lemeit/lemeit-wq)

## Origin and status (August 2026)

It existed as a loose file (`docs/agua_saladillo.html`) inside the `ema-saladillo` repo; it was migrated to its own repo in August 2026 to evolve independently, like the other two sibling projects.

Unlike Air and EMA, **it doesn't have a public data API yet**: the samples (`RAW`) and the regulatory limits (`LIM`) are still embedded as a JS object inside `index.html` itself, with no database or backend of its own for that part — it's the main item on the pending roadmap. What does have its own backend is the **sampling points' coordinates**, served from Cloudflare D1 via a Worker (same pattern as the other two portals).

## Data origin

The values come from the testing protocols the Municipality of Saladillo publishes as loose PDFs at [saladillo.gob.ar/servicios_sanitarios](https://www.saladillo.gob.ar/servicios_sanitarios) — with no table, index, or consistent file names. The initial load (87 samples) was manual, protocol by protocol. Since August 2026 there's a GitHub Action (`protocolos-ingest.yml`, triggered manually) that downloads new PDFs and uses the Gemini API to extract structured JSON from each one — the result lands in a staging CSV (`extraidos_pendientes.csv`) for human review before merging into the dashboard, never directly.

## Existing API (partial)

| Endpoint | Description |
|---|---|
| `GET /api/coords` | Sampling point coordinates — public, read-only |
| `POST /api/coords` | Edit coordinates — protected with an `X-Admin-Key` header (401 without the correct key) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Map tile proxy to CARTO Basemaps |

Base URL: `https://api.lemeit.ar/wq` (public domain shared with the other environmental-network projects, via a Cloudflare Route — same scheme as `api.lemeit.ar/aq` and `api.lemeit.ar/emas`; the original `*.workers.dev` — `agua-saladillo-api...workers.dev` — still works the same as a direct fallback, without the `/wq` prefix). `app.lemeit.ar/wq` serves the static dashboard, not the API (the old `wq.lemeit.ar` just redirects there).

### Usage example

```bash
curl "https://api.lemeit.ar/wq/api/coords"
```

The samples (`RAW`) and regulatory limits (`LIM`) don't have an API yet — today the only way to access them is by reading the JS object embedded in the dashboard's `index.html`; there's no separate endpoint to request them.

## A note on Arsenic

It's the only parameter with a real discrepancy between regulations: the Argentine Food Code sets 0.01 mg/L (adopted from the WHO) while the current text of Provincial Law 11,820 (Annex A) still says 0.05 mg/L, never updated — although in practice the Province follows the WHO/CAA value, which is also what the municipal protocols themselves cite. The dashboard shows **both** limits instead of picking one, to make the gap between the written rule and actual practice visible.

## Pending roadmap

- Own backend for `RAW`/`LIM` (Cloudflare D1 + Worker), following the same pattern already used by `lemeit-emas` and `lemeit-aq` — the step that would enable a public data API just like the other two portals, and let automatic ingestion write straight to the database instead of to a staging CSV.

See the [project log](99-bitacora.md) (Spanish only) for more context on harmonizing the three portals.

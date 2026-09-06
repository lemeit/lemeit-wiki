# ![AQ](../assets/logos/aq.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Aire Saladillo — aq.lemeit.ar

Red de sensores de calidad del aire (PM1.0/PM2.5/PM10, VOC, CO2, NOx, temperatura, humedad, presión) instalada en escuelas, jardines de infantes y domicilios de Saladillo. Combina sensores **PurpleAir** y **AirGradient** en una misma base y un mismo dashboard.

Repositorio: [github.com/lemeit/purpleair-saladillo](https://github.com/lemeit/purpleair-saladillo)

## Arquitectura

```
PurpleAir API + AirGradient API
        ↓ (ingesta cada 15 min)
Cloudflare Cron Trigger (Worker) + GitHub Actions (respaldo) + cron-job.org (respaldo externo)
        ↓
Cloudflare D1 — tablas "sensores" (metadata) y "lecturas" (serie temporal)
        ↓
Worker "purpleair-saladillo-api" — API REST propia (JSON/CSV)
        ↓
index.html (Cloudflare Pages) — dashboard estático
```

Triple redundancia de ingesta deliberada: si el Cron Trigger de Cloudflare falla en silencio (bug conocido de la plataforma), GitHub Actions y cron-job.org cubren el hueco. El `INSERT OR IGNORE` + `UNIQUE(sensor_index, timestamp)` en `lecturas` evita duplicados aunque los tres corran en paralelo.

## API pública

Sin autenticación, CORS abierto, pensada para que cualquiera la consuma directo — no solo el propio dashboard. También hay documentación interactiva con ejemplos en [aq.lemeit.ar/api.html](https://aq.lemeit.ar/api.html).

| Endpoint | Descripción |
|---|---|
| `GET /api/sensores` | Metadata de todos los sensores activos |
| `GET /api/ultimas` | Última lectura de cada sensor |
| `GET /api/historico/:sensor_index` | Historial de un sensor — `range=24h\|7d\|30d`, o `desde`/`hasta` (rango de fechas absoluto en UTC), `limit` (máx. 20000) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

Los tres primeros aceptan `&formato=csv` para descargar CSV en vez de JSON.

## Guía de uso de la API

Base URL: `https://aq.lemeit.ar`. Todos los ejemplos funcionan pegados directo en la barra del navegador o con `curl`.

**Metadata de los sensores activos:**

```bash
curl "https://aq.lemeit.ar/api/sensores"
```

**Última lectura de cada sensor, en CSV:**

```bash
curl "https://aq.lemeit.ar/api/ultimas?formato=csv" -o ultimas.csv
```

**Histórico de un sensor puntual, últimos 7 días:**

```bash
curl "https://aq.lemeit.ar/api/historico/12345?range=7d"
```

**Mismo sensor, rango de fechas absoluto (UTC) y en CSV:**

```bash
curl "https://aq.lemeit.ar/api/historico/12345?desde=2026-08-01&hasta=2026-08-31&formato=csv" -o agosto.csv
```

`desde`/`hasta` aceptan `YYYY-MM-DD` (toma desde el inicio/fin de ese día en UTC) o `YYYY-MM-DD HH:MM:SS` para precisión horaria. Cuando están presentes, reemplazan a `range`.

**Desde JavaScript (por ejemplo, para un dashboard propio):**

```javascript
const resp = await fetch("https://aq.lemeit.ar/api/ultimas");
const sensores = await resp.json();
```

**Desde Python + pandas, directo a un DataFrame:**

```python
import pandas as pd
df = pd.read_csv("https://aq.lemeit.ar/api/historico/12345?range=30d&formato=csv")
```

El `sensor_index` de cada sensor sale de `GET /api/sensores` — no hay que adivinarlo.

## Hitos técnicos

- **Deduplicación**: cuando un sensor pierde conexión, PurpleAir sigue devolviendo su último `last_seen` congelado — sin `UNIQUE(sensor_index, timestamp)` cada corrida del cron generaba una fila nueva con el mismo timestamp, produciendo tarjetas duplicadas/triplicadas en el dashboard.
- **AirGradient (agosto 2026)**: se sumó como segundo proveedor de sensores, reutilizando las mismas tablas (`proveedor` distingue el origen) en vez de un portal separado.
- **Proxy de tiles**: CARTO empezó a exigir API key para servir tiles; en vez de exponerla en el HTML público, el Worker actúa de proxy y agrega la key del lado del servidor (`CARTO_API_KEY` como secret).
- **API pública (agosto 2026)**: los endpoints de lectura, que ya existían para alimentar el propio dashboard, se documentaron y ampliaron (rango de fechas absoluto, export CSV) para que terceros —por ejemplo, otro sector de la escuela o de la Municipalidad— puedan consumir los datos sin depender del dashboard.
- **Admin básico de visitas (septiembre 2026)**: nueva tabla `visitas` en D1 (escritura en segundo plano vía `ctx.waitUntil`, sin demorar la respuesta real) y dos endpoints de solo lectura (`/api/admin/visitas`, `/api/admin/resumen`) protegidos por un secret `ADMIN_KEY` enviado como header `X-Admin-Key` (no como `?key=...` en la URL, para no dejarlo pegado en el historial del navegador ni en logs) — mismo esquema que ya usaba `X-Ingest-Key`. Se sumó `scripts/ver-visitas.ps1` para consultarlo desde PowerShell sin pelearse con la sintaxis de headers de `curl.exe`.
- **Vista "Reporte" en Historial y exportación a PDF (septiembre 2026)**: toggle Clásica/Reporte en la tabla de Historial — la vista Reporte imita el formato de texto plano/fuente monoespaciada de los resúmenes climatológicos de estaciones meteorológicas (estilo NOAA), con fondo papel fijo independiente del tema claro/oscuro del sitio. Se sumaron filtro de rango de fechas personalizado, checkboxes para mostrar/ocultar columnas, y descarga a PDF con texto real (no una captura de pantalla) vía jsPDF, con paginación automática. jsPDF se carga desde cdnjs con respaldo a jsDelivr si falla, y un aviso claro al usuario si ninguna de las dos CDN está disponible.

## Roadmap

- **Expansión a 4 sensores nuevos** (en evaluación, agosto 2026): 2 en Saladillo y 2 en el partido de **25 de Mayo** — en cada partido, una escuela de zona urbana y una de zona rural. Es la primera vez que la red de aire deja de ser exclusivamente de Saladillo.
- Esta expansión abre la puerta a un futuro **apartado de reportes** que combine datos de Aire Saladillo con los de [EMA](02-ema-saladillo.md) (meteorología) y análisis espacial entre sensores — hoy no es inmediato porque las estaciones EMA y los sensores de aire no comparten ubicación física.
- La ampliación a 25 de Mayo también empuja a que la red meteorológica **EMA** sume una estación en ese partido (ver el roadmap de [EMA Saladillo](02-ema-saladillo.md)), para que ambas redes cubran la misma región.

Ver la [Bitácora del proyecto](99-bitacora.md) para el detalle sesión por sesión.

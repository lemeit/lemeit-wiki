# Aire Saladillo — aq.lemeit.ar

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

Sin autenticación, CORS abierto, pensada para que cualquiera la consuma directo. Documentación con ejemplos: [aq.lemeit.ar/api.html](https://aq.lemeit.ar/api.html).

| Endpoint | Descripción |
|---|---|
| `GET /api/sensores` | Metadata de todos los sensores activos |
| `GET /api/ultimas` | Última lectura de cada sensor |
| `GET /api/historico/:sensor_index` | Historial de un sensor — `range=24h\|7d\|30d`, o `desde`/`hasta` (rango de fechas absoluto en UTC), `limit` (máx. 20000) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

Los tres primeros aceptan `&formato=csv` para descargar CSV en vez de JSON.

## Hitos técnicos

- **Deduplicación**: cuando un sensor pierde conexión, PurpleAir sigue devolviendo su último `last_seen` congelado — sin `UNIQUE(sensor_index, timestamp)` cada corrida del cron generaba una fila nueva con el mismo timestamp, produciendo tarjetas duplicadas/triplicadas en el dashboard.
- **AirGradient (agosto 2026)**: se sumó como segundo proveedor de sensores, reutilizando las mismas tablas (`proveedor` distingue el origen) en vez de un portal separado.
- **Proxy de tiles**: CARTO empezó a exigir API key para servir tiles; en vez de exponerla en el HTML público, el Worker actúa de proxy y agrega la key del lado del servidor (`CARTO_API_KEY` como secret).
- **API pública (agosto 2026)**: los endpoints de lectura, que ya existían para alimentar el propio dashboard, se documentaron y ampliaron (rango de fechas absoluto, export CSV) para que terceros —por ejemplo, otro sector de la escuela o de la Municipalidad— puedan consumir los datos sin depender del dashboard.

Ver la [Bitácora del proyecto](99-Bitacora.md) para el detalle sesión por sesión.

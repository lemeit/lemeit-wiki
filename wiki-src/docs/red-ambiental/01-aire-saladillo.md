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

## Roadmap

Se planean 4 sensores nuevos: 2 en Saladillo y 2 en 25 de Mayo (un establecimiento urbano y uno rural en cada partido), ampliando la red más allá del partido de Saladillo por primera vez. Ver también el roadmap de [EMA Saladillo](02-ema-saladillo.md#roadmap), que suma una estación en 25 de Mayo por el mismo motivo.

## Hitos técnicos

- **Deduplicación**: cuando un sensor pierde conexión, PurpleAir sigue devolviendo su último `last_seen` congelado — sin `UNIQUE(sensor_index, timestamp)` cada corrida del cron generaba una fila nueva con el mismo timestamp, produciendo tarjetas duplicadas/triplicadas en el dashboard.
- **AirGradient (agosto 2026)**: se sumó como segundo proveedor de sensores, reutilizando las mismas tablas (`proveedor` distingue el origen) en vez de un portal separado.
- **Proxy de tiles**: CARTO empezó a exigir API key para servir tiles; en vez de exponerla en el HTML público, el Worker actúa de proxy y agrega la key del lado del servidor (`CARTO_API_KEY` como secret).
- **API pública (agosto 2026)**: los endpoints de lectura, que ya existían para alimentar el propio dashboard, se documentaron y ampliaron (rango de fechas absoluto, export CSV) para que terceros —por ejemplo, otro sector de la escuela o de la Municipalidad— puedan consumir los datos sin depender del dashboard.
- **Administración de visitas (septiembre 2026)**: el Worker registra visitas al dashboard; las consultas administrativas pasaron de clave por query param a header `X-Admin-Key` (evita que la clave quede en logs de acceso/proxies intermedios). Se sumó `scripts/ver-visitas.ps1` para consultarlas desde PowerShell sin pasar por un navegador.
- **Historial — reporte configurable y export PDF (septiembre 2026)**: se agregó filtro de rango de fechas, filtro de parámetros (incluyendo PM1.0/PM10, que PurpleAir y AirGradient ya reportaban pero no se mostraban) y exportación a PDF con formato de reporte climatológico estilo NOAA — texto plano monoespaciado (fuente Courier de jsPDF, sin embeber archivos de fuente), siempre en fondo blanco, independiente del tema claro/oscuro del resto del sitio. Al generarse, el PDF además se abre en una pestaña nueva (blob URL, visor nativo del navegador) al mismo tiempo que se descarga. jsPDF carga desde cdnjs con respaldo automático a jsDelivr si el CDN principal falla (bloqueadores de contenido, cortes de red).
  - La vista "Reporte" convivió un tiempo como toggle en pantalla junto a la tabla normal ("Clásica"); se simplificó sacando el toggle — la tabla en pantalla usa siempre el formato normal del sitio, y el estilo "reporte" quedó reservado exclusivamente al PDF, que de todos modos ya se veía siempre en blanco.
  - Terminología: "Estación" se reemplazó por "Sitio" (encabezado del PDF y demás lugares que nombran la ubicación de un sensor), para no chocar con "estación meteorológica" del proyecto hermano EMA.
- **Gráfico — comparación de sitios (septiembre 2026)**: nuevo modo donde se elige un parámetro (no varios, para no mezclar unidades en un mismo eje) y varios sitios a la vez — cada sitio se dibuja como una línea de color distinto sobre el mismo eje Y. Al no medir todos los sitios en el mismo instante exacto (cada uno hace su propio polling), el eje de tiempo se arma uniendo las marcas de todos los sitios elegidos, con `spanGaps` para no cortar las líneas en los huecos propios de cada uno.
- **Enlaces a la wiki**: el footer compartido (`lemeit-common.js`, usado por los tres portales) y la sección "Acerca de" de cada uno enlazan a [wiki.lemeit.ar](https://wiki.lemeit.ar).

Ver la [Bitácora del proyecto](99-bitacora.md) para el detalle sesión por sesión.

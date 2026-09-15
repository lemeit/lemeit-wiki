# ![AQ](../assets/logos/aq.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Monitoreo Ambiental Escolar — aq.lemeit.ar

Red de sensores de calidad del aire (PM1.0/PM2.5/PM10, VOC, CO2, NOx, temperatura, humedad, presión) instalada en instituciones educativas y domicilios de la Provincia de Buenos Aires — proyecto piloto en los partidos de Saladillo y 25 de Mayo. Combina sensores **PurpleAir** y **AirGradient** en una misma base y un mismo dashboard.

Repositorio: [github.com/lemeit/purpleair-saladillo](https://github.com/lemeit/purpleair-saladillo)

## Arquitectura

```
PurpleAir API + AirGradient API
        ↓ (ingesta cada 15 min)
cron-job.org (externo, único camino activo) — GitHub Actions y Cron Trigger de Cloudflare quedan en reserva
        ↓
Cloudflare D1 — tablas "sensores" (metadata) y "lecturas" (serie temporal)
        ↓
Worker "purpleair-saladillo-api" — API REST propia (JSON/CSV)
        ↓
index.html (Cloudflare Pages) — dashboard estático
```

Hasta agosto 2026 corrían en paralelo tres disparadores de ingesta (Cron Trigger de Cloudflare, GitHub Actions y cron-job.org) como redundancia deliberada ante el bug de Cloudflare que hace que el Cron Trigger quede registrado pero nunca dispare. El `INSERT OR IGNORE` + `UNIQUE(sensor_index, timestamp)` en `lecturas` evita filas duplicadas, pero no evita que cada camino activo gaste su propia llamada a la API de PurpleAir — con 4 sensores nuevos por sumarse, en septiembre 2026 se pausó GitHub Actions (queda `workflow_dispatch` para correrlo a mano) y **cron-job.org pasó a ser el único disparador activo**, para no pagar doble en puntos de la API. Detalle completo en el [README del repo](https://github.com/lemeit/purpleair-saladillo#readme).

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

Base URL: `https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev` (el Worker de la API — `aq.lemeit.ar` sirve el dashboard estático, no la API; el propio `index.html` y `api.html` del repo usan esta misma base). Todos los ejemplos funcionan pegados directo en la barra del navegador o con `curl`.

**Metadata de los sensores activos:**

```bash
curl "https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/sensores"
```

**Última lectura de cada sensor, en CSV:**

```bash
curl "https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/ultimas?formato=csv" -o ultimas.csv
```

**Histórico de un sensor puntual, últimos 7 días:**

```bash
curl "https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/historico/12345?range=7d"
```

**Mismo sensor, rango de fechas absoluto (UTC) y en CSV:**

```bash
curl "https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/historico/12345?desde=2026-08-01&hasta=2026-08-31&formato=csv" -o agosto.csv
```

`desde`/`hasta` aceptan `YYYY-MM-DD` (toma desde el inicio/fin de ese día en UTC) o `YYYY-MM-DD HH:MM:SS` para precisión horaria. Cuando están presentes, reemplazan a `range`.

**Desde JavaScript (por ejemplo, para un dashboard propio):**

```javascript
const resp = await fetch("https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/ultimas");
const sensores = await resp.json();
```

**Desde Python + pandas, directo a un DataFrame:**

```python
import pandas as pd
df = pd.read_csv("https://purpleair-saladillo-api.fisicai-eureka-01.workers.dev/api/historico/12345?range=30d&formato=csv")
```

El `sensor_index` de cada sensor sale de `GET /api/sensores` — no hay que adivinarlo.

## Scripts de administración

Dos scripts de PowerShell en `scripts/` del repo, para consultas rápidas por consola sin abrir el navegador (los endpoints que usan van por header, no se pueden pegar directo en la barra de direcciones).

**Ver visitas al sitio** (`scripts/ver-visitas.ps1`) — consulta los endpoints admin del propio Worker (`/api/admin/visitas`, `/api/admin/resumen`), protegidos por el secret `ADMIN_KEY`:

```powershell
cd scripts
.\ver-visitas.ps1                    # resumen: 24h, 7d, top rutas, top países
.\ver-visitas.ps1 -Modo visitas      # últimas 200 visitas, una por una
.\ver-visitas.ps1 -Modo visitas -Limit 50
```

Pide la `ADMIN_KEY` por consola (oculta al tipear), o se puede dejar puesta para toda la sesión de PowerShell con `$env:PA_ADMIN_KEY = "tu_clave"` así no se vuelve a pedir.

**Ver uso de la API de PurpleAir** (`scripts/ver-uso-purpleair.ps1`) — consulta directo contra `api.purpleair.com` (no pasa por nuestro Worker) cuántos puntos quedan y a qué ritmo se consumen:

```powershell
cd scripts
.\ver-uso-purpleair.ps1
```

Pide la `PURPLEAIR_API_KEY` (la misma que se cargó con `wrangler secret put`), o se puede fijar con `$env:PA_API_KEY = "tu_clave"` para no re-tipearla. Esta consulta **no gasta puntos de la cuota** — PurpleAir la marca explícitamente como "a free API call, so query it as you need" (confirmado por su staff en el foro oficial de la API, abril 2026), así que se puede correr las veces que haga falta.

## Roadmap

**Estado del hardware (septiembre 2026)**: los 5 sensores **PurpleAir** ya están en poder del proyecto, pero hoy solo 1 está en funcionamiento — el del Colegio Secundario Madre Teresa, pendiente de reubicación/reinstalación. Los otros 4 se están instalando ahora en el resto de las instituciones de Saladillo y en las de 25 de Mayo. Los 2 sensores **AirGradient** también están en mano y activos, hoy conectados en un domicilio particular a modo de prueba, a la espera de definir en qué instituciones se instalan.

Se planean 4 sensores nuevos (en evaluación, agosto 2026): 2 en Saladillo y 2 en el partido de **25 de Mayo** — en cada partido, una escuela de zona urbana y una de zona rural, ampliando la red más allá del partido de Saladillo por primera vez. Ver también el roadmap de [EMA Saladillo](02-ema-saladillo.md#roadmap), que suma una estación en 25 de Mayo por el mismo motivo. Esta expansión abre la puerta a un futuro apartado de reportes combinados (aire + meteorología) con análisis espacial entre sensores — hoy no es inmediato porque las estaciones EMA y los sensores de aire no comparten ubicación física.

## Hitos técnicos

- **Deduplicación**: cuando un sensor pierde conexión, PurpleAir sigue devolviendo su último `last_seen` congelado — sin `UNIQUE(sensor_index, timestamp)` cada corrida del cron generaba una fila nueva con el mismo timestamp, produciendo tarjetas duplicadas/triplicadas en el dashboard.
- **AirGradient (agosto 2026)**: se sumó como segundo proveedor de sensores, reutilizando las mismas tablas (`proveedor` distingue el origen) en vez de un portal separado.
- **Proxy de tiles**: CARTO empezó a exigir API key para servir tiles; en vez de exponerla en el HTML público, el Worker actúa de proxy y agrega la key del lado del servidor (`CARTO_API_KEY` como secret).
- **API pública (agosto 2026)**: los endpoints de lectura, que ya existían para alimentar el propio dashboard, se documentaron y ampliaron (rango de fechas absoluto, export CSV) para que terceros —por ejemplo, otro sector de la escuela o de la Municipalidad— puedan consumir los datos sin depender del dashboard.
- **Administración de visitas (septiembre 2026)**: nueva tabla `visitas` en D1, escrita en segundo plano vía `ctx.waitUntil` para no demorar la respuesta real, y dos endpoints de solo lectura (`/api/admin/visitas`, `/api/admin/resumen`) protegidos por un secret `ADMIN_KEY` enviado como header `X-Admin-Key` — mismo esquema que ya usaba `X-Ingest-Key` — en vez de `?key=...` en la URL, para no dejar la clave pegada en el historial del navegador ni en logs de acceso/proxies intermedios. Se sumó `scripts/ver-visitas.ps1` para consultarlo desde PowerShell, ya que ahí `curl` es un alias de `Invoke-WebRequest` con una sintaxis de headers distinta a la del `curl` real (hace falta `curl.exe` o `-Headers @{...}`).
- **Historial — reporte configurable y export PDF (septiembre 2026)**: se agregó filtro de rango de fechas personalizado (`desde`/`hasta`), filtro de parámetros por checkboxes (incluyendo PM1.0/PM10, que PurpleAir y AirGradient ya reportaban pero no se mostraban) y exportación a PDF con formato de reporte climatológico estilo NOAA — texto plano monoespaciado (fuente Courier de jsPDF, sin embeber archivos de fuente), siempre en fondo blanco, independiente del tema claro/oscuro del resto del sitio. jsPDF se carga desde cdnjs con respaldo automático a jsDelivr si el CDN principal falla (se vio en producción: un bloqueador de contenido impedía cargar el script y tiraba `window.jspdf` indefinido, sin manejar la excepción — ahora hay un aviso claro al usuario en vez de romper la página). El PDF, además de descargarse, ahora también se abre en una pestaña nueva (blob URL, visor nativo del navegador) dentro del mismo click, para poder verlo sin ir a buscar el archivo descargado.
    - La vista "Reporte" en pantalla (toggle Clásica/Reporte junto a la tabla normal) resultó redundante tras uso real — el PDF ya se veía siempre en blanco/formato reporte. Se sacó el toggle: la tabla en pantalla usa siempre el formato normal del sitio, y el estilo reporte quedó reservado al PDF. De paso se agregó `accent-color` a los checkboxes del sitio (el azul por defecto del navegador desentonaba con la paleta).
    - Terminología: "Estación" se reemplazó por "Sitio" (encabezado del PDF y demás lugares que nombran la ubicación de un sensor), para no chocar con "estación meteorológica" del proyecto hermano EMA.
- **Gráfico — comparación de sitios (septiembre 2026)**: nuevo modo ("Comparar sitios") donde se elige un parámetro — no varios, para no mezclar unidades en un mismo eje — y varios sitios a la vez, cada uno dibujado como una línea de color distinto sobre el mismo eje Y. Al no medir todos los sitios en el mismo instante exacto (cada uno hace su propio polling), el eje de tiempo se arma uniendo las marcas de todos los sitios elegidos, con `spanGaps` para no cortar las líneas en los huecos propios de cada uno — una limitación conocida, a mejorar a futuro con un agrupado por intervalos redondeados.
- **PWA instalable (septiembre 2026)**: el dashboard se puede instalar como app (`manifest.json`, ícono "maskable" para Android, `sw.js`). Al abrirse instalado — o con `?pwa=1` en la URL, que es el `start_url` del manifest — arranca directo en la pestaña "Actuales" en vez de "Mapa": los relojes tipo velocímetro de AQI son la portada más útil en el celular. Detección de lanzamiento vía `matchMedia("(display-mode: standalone)")` más `navigator.standalone` (cubre Safari/iOS, que no soporta bien esa media query). El service worker cachea solo el "app shell" (HTML, manifest, íconos); los datos en vivo (API propia) y las librerías de CDN (Chart.js, jsPDF, Leaflet, fuentes, design.lemeit.ar) van siempre directo a red, para no quedar con lecturas ni versiones de librerías desactualizadas.
- **Gráfico — zoom por selección (septiembre 2026)**: se sumó `chartjs-plugin-zoom` (+ `hammerjs` para gestos táctiles) al Gráfico, en ambos modos (histórico de un sitio y "Comparar sitios"): arrastre del mouse para zoom por selección ("box zoom"), rueda para acercar/alejar, pellizco táctil en celular, y un botón "Reset zoom" que aparece solo cuando hay zoom aplicado. Mismo patrón de doble CDN con respaldo (jsDelivr primero, cdnjs si falla) ya usado para jsPDF.
    - **Fix**: en el primer despliegue, arrastrar para seleccionar terminaba haciendo zoom hacia el lado contrario de donde se arrastraba, y peor cuantos más zooms anidados (zoom sobre un zoom ya aplicado). Causa: el paneo (mover la vista arrastrando) y el zoom por selección estaban atados al mismo gesto — arrastre simple, sin modificador — y competían entre sí; el paneo corría el eje mientras se armaba el rectángulo de selección. Se resolvió exigiendo Ctrl+arrastre para panear, dejando el arrastre simple exclusivamente para la selección de zoom. De paso se cambió el eje X de una escala de categorías (labels de texto) a una escala numérica (`linear`, con los horarios mostrados vía un callback de ticks), porque `chartjs-plugin-zoom` no soporta bien los zooms anidados sobre un eje de categorías.
- **Enlaces a la wiki**: el footer compartido (`lemeit-common.js`, usado por los tres portales) y la sección "Acerca de" de cada uno pasan a enlazar a [wiki.lemeit.ar](https://wiki.lemeit.ar) — hasta ahora la wiki no se mencionaba desde ningún portal.

Ver la [Bitácora del proyecto](99-bitacora.md) para el detalle sesión por sesión.

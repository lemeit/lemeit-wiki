# Bitácora del proyecto

Historial de desarrollo de los tres portales, reconstruido a partir de las bitácoras originales de EMA Saladillo (`docs/bitacora_ema_saladillo*.docx`, hoy migradas acá) y del historial de commits de los tres repositorios. Desarrollado en sesiones colaborativas con Claude (Anthropic).

## Marzo 2026 — Origen: EMA Saladillo

### Contexto

El proyecto nace en el espacio curricular **Laboratorio de Industrias**, 7° Año Técnico Químico, **EEST N°1 "Gral. Savio"** (Saladillo), a cargo del Ing. Luciano Lamaita. La EEST N°1 tiene una Estación Meteorológica Automática (EMA) marca Tecmes que transmite por telemetría al Sistema Nacional de Información Hídrica (SNIH) del INA — el objetivo inicial fue acceder a esos datos de forma programática, independiente del sitio oficial. Durante el desarrollo se descubrieron tres estaciones adicionales con datos públicos en Saladillo (CFR, Defensa Civil, Clima Saladillo), lo que amplió el alcance hacia una red meteorológica local con análisis espacial.

### v1 — Arquitectura inicial

Patrón ETL corrido localmente en una PC Windows: scrapers Python → Supabase (PostgreSQL) → dashboard HTML estático. Cada estación con un método de acceso distinto:

- **EET N°1**: `POST` a `MuestraDatos.aspx/LeerDatosActuales` — el certificado SSL del servidor está vencido (`verify=False`), y el ID de estación correcto (`284094`) resultó ser la concatenación del código de red RMET (`28`) con el ID real (`4094`) — llamar solo con `4094` devolvía vacío.
- **CFR Saladillo**: scraping HTML con BeautifulSoup.
- **Defensa Civil**: sin ningún endpoint de datos — toda la información está superpuesta como texto sobre una imagen JPG de cámara Meteobridge. Se resolvió con recorte + OCR (Tesseract), ~95% de precisión.
- **Clima Saladillo**: los datos se cargan por AJAX; el endpoint oculto (`stationDataAjax.php`) se descubrió inspeccionando el JavaScript de la página.

Deduplicación: `hora_ar()`, una función PostgreSQL `IMMUTABLE` que trunca el timestamp a la hora, permite un índice único y evita duplicados por múltiples ejecuciones del scraper.

### 21–22 de marzo — v2: cuarta estación y análisis de calidad de datos

- Se suma **EMA-CS** (Clima Saladillo, barrio Falucho) y se establece la nomenclatura oficial `EMA-XXX` en todo el sistema.
- **Windows Task Scheduler** fallaba con código `0x80070002` en todas las tareas: el Programador de Tareas corre en un contexto donde el `PATH` no incluye la instalación de Python del usuario — se resolvió usando la ruta completa del ejecutable.
- **Diagnóstico y limpieza de EMA-DC**: los errores sistemáticos venían del OCR — la conversión a escala de grises pierde el texto blanco sobre fondos de color. Se resolvió extrayendo directamente los píxeles claros (`R,G,B > 175`) antes de binarizar, más una validación en dos capas (rango físico plausible + delta temporal máximo) que descarta valores imposibles antes de insertar. Se eliminaron a mano 5 temperaturas y 3 valores de lluvia erróneos ya acumulados.
- **Armonización temporal**: nace la vista `v_ema_armonizada` (normaliza las 4 fuentes a una estructura común por hora) y `v_temperatura_comparativa` (pivotada, una columna por estación).
- **Análisis microclimático**: con los primeros datos se observa que EMA-CS marca sistemáticamente +2 a +4°C de noche y -1 a -2°C de día respecto al promedio de las otras tres — hipótesis de **isla de calor urbano** (el barrio Falucho es zona urbana consolidada). También se detecta un offset de ~8 hPa en la presión de EMA-EET, sin explicación confirmada (¿presión absoluta vs. reducida a nivel del mar?).
- Dashboard v4: pestaña de análisis espacial con interpolación **IDW** (Inverse Distance Weighting, p=2) e isolíneas por **Marching Squares**; modo comparativo rediseñado con tabla de diferencias por color.

### 23 de marzo — v3: GitHub Actions y primer deploy público

- **Migración de Windows Task Scheduler a GitHub Actions**: el sistema dependía de una PC física encendida. Repo público creado (`github.com/lemeit/ema-saladillo`); credenciales de Supabase movidas a Secrets de GitHub (`SUPA_URL`, `SUPA_KEY`).
- Dos cron schedules: scrapers horarios (`5 * * * *`) y ping diario a Supabase a las 11:05 UTC / 08:05 AR (evita que el plan gratuito pause el proyecto por inactividad).
- Corrección de coordenadas: EMA-CFR y EMA-DC tenían las coordenadas intercambiadas desde el inicio del proyecto.
- Primer dashboard público: `lemeit.github.io/ema-saladillo` (GitHub Pages, dashboard movido a `docs/index.html` por la convención de esa plataforma).

## Agosto 2026 — Nace Aire Saladillo y se arma la red de 3 portales

### 19–20 de agosto — Aire Saladillo (aq.lemeit.ar)

Nuevo proyecto, `purpleair-saladillo`: sensores PurpleAir de calidad del aire (PM1.0/PM2.5/PM10) en escuelas y jardines, sobre **Cloudflare D1 + Workers + Pages** desde el día uno (arquitectura más moderna que la de EMA en ese momento, que todavía corría sobre Supabase). Rango temporal, bandas AQI, canales A/B de PM y export CSV desde el inicio.

### 21–24 de agosto — Diseño unificado y agua-saladillo

- **Etapa 1 de diseño unificado**: nace [design.lemeit.ar](https://design.lemeit.ar) (`lemeit-theme.css` + `lemeit-common.js`) — paleta, tipografía (JetBrains Mono), selector de portales y footer versionado compartidos entre aq y ema.
- Pestaña **Mapa** (Leaflet) con flip-cards y mini-gráfico en aq; canales A/B extendidos a PM1.0/PM10 (no solo PM2.5).
- **Migración de `agua-saladillo`**: el dashboard de calidad del agua, que vivía como archivo suelto (`docs/agua_saladillo.html`) dentro de `ema-saladillo`, se muda a repo propio para evolucionar de forma independiente — tercer portal de la red.
- `agua-saladillo` incorpora dataset completo (31 → 52 parámetros tras un merge posterior), normativa comparada CAA/PBA, y un primer GitHub Action de ingesta automática de protocolos PDF usando la API de Gemini para extraer JSON estructurado (con revisión humana obligatoria antes de mergear — los protocolos no tienen un formato de tabla único, así que la extracción declara su propio nivel de confianza).

### 21–22 de agosto — Armonización de EMA y migración a Cloudflare D1 (v4)

Sesión de alcance ampliado: unificar los tres proyectos sobre la misma infraestructura de Cloudflare.

- **Cloudflare Pages para EMA**: dashboard movido de `docs/index.html` (convención GitHub Pages) a la raíz del repo (Cloudflare Pages no tiene esa restricción); publicado en `emas.lemeit.ar`. El deploy es manual vía `wrangler pages deploy` — a diferencia de `index.html`/`api.html`, **no hay deploy automático por push** en este proyecto.
- **Migración Supabase → Cloudflare D1**: se exportó el historial completo (30.213 filas de las 4 tablas) y se importó a una tabla unificada `mediciones` (columna `estacion`). Se descartaron 9 filas de EMA-DC por un timestamp con error de OCR confirmado como dato corrupto real.
- **Worker de compatibilidad** (`ema-saladillo-api`): expone D1 con el mismo formato de consulta que usaba PostgREST/Supabase (`select=`, `order=`, `limit=`, `columna=eq.valor`) — el único cambio necesario en el dashboard fue apuntar `SUPA_URL` al Worker, sin tocar la lógica de consultas.
- **Scrapers actualizados**: los 4 scripts pasan a escribir en D1 vía un helper común (`scrapers/d1_writer.py`), mismo patrón que ya usaba `purpleair-saladillo`. Se detectaron y corrigieron 3.796 filas duplicadas en el historial migrado (el `ignore-duplicates` de Supabase nunca había funcionado realmente, porque las tablas viejas no tenían restricción única) y un `403 Forbidden` causado por saltos de línea de sobra en los secrets de Cloudflare al pegarlos en GitHub (`%0A` en la URL de la API) — resuelto con `.strip()`.
- El workflow de scrapers se había roto en una sesión anterior de "armonización de repositorios" (el archivo terminó en `github/workflows/`, sin el punto) — restaurado a su ruta correcta.

### 23–28 de agosto — Paridad visual, AirGradient, mapas

- Header y tabs homogeneizados entre los tres portales; footer compartido con logos de proveedor/institución.
- **AirGradient** se suma como segundo proveedor de sensores en Aire Saladillo (además de PurpleAir), reutilizando las mismas tablas D1 (columna `proveedor`). Se agrega CO2 y luego NOx al dashboard.
- **Proxy de tiles CARTO**: CARTO empezó a exigir API key para servir tiles de mapa; en vez de exponerla en el HTML público, los tres Workers pasan a actuar de proxy, agregando la key del lado del servidor (`CARTO_API_KEY` como secret de Wrangler) — patrón replicado igual en los tres portales el mismo día (28/08).
- Modo claro por defecto en los tres portales (antes dependía del tema del sistema operativo).
- Fix de overflow-x (el mapa Leaflet arrastraba toda la página al costado al hacer pan) — aplicado a los tres portales.

### 29–31 de agosto — API pública, fixes y esta wiki

- **Fix**: `scrapers/supabase_ping.py` (EMA) tenía un `NameError` — `SUPA_URL`/`SUPA_KEY` solo existían como valores anidados dentro del diccionario `PROYECTOS`, nunca como nombres a nivel de módulo, lo que hacía fallar el job `ping` de GitHub Actions apenas arrancaba.
- **Aire Saladillo**: Mapa pasa a ser la pestaña por defecto (antes era Actuales); fix de un bug de apilamiento donde el dropdown "Portales" quedaba parcialmente tapado por el mapa Leaflet (los paneles internos de Leaflet tienen `z-index` de hasta 650–700 y, sin un contexto de apilamiento propio en `#mapa-container`, competían directamente contra el header) — resuelto dándole al contenedor del mapa su propio `position: relative; z-index: 1`.
- **API pública en Aire Saladillo y EMA Saladillo**: los endpoints de lectura que ya alimentaban cada dashboard (`/api/sensores`, `/api/ultimas`, `/api/historico` en aq; `/rest/v1/mediciones_*` y las vistas armonizadas en ema) se documentaron y ampliaron para uso de terceros — sin necesidad de infraestructura nueva, ya eran de lectura pública con CORS abierto:
    - `desde`/`hasta` como rango de fechas absoluto en UTC, alternativa a la ventana relativa (`range`/`horas`) que ya existía.
    - `&formato=csv` en todos los endpoints de lectura, para descargar CSV directo sin parsear JSON.
    - Página de documentación (`api.html`) publicada en cada portal — [aq.lemeit.ar/api.html](https://aq.lemeit.ar/api.html) y [emas.lemeit.ar/api.html](https://emas.lemeit.ar/api.html).
- **Corrección de dominio**: se detectó y corrigió una referencia equivocada a `ema.lemeit.ar` (dominio inexistente) en vez del dominio real `emas.lemeit.ar`, introducida en la documentación nueva de la API de EMA.
- **Esta wiki**: se arma `ambiental-wiki`, siguiendo el mismo patrón que ya usa [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/` con MkDocs Material, build automático vía GitHub Actions), para tener documentación técnica y bitácora unificadas de los tres portales en un solo lugar — hasta acá solo existía para EMA, como 4 documentos Word sueltos (`bitacora_ema_saladillo_v1..v4.docx`, migrados acá).

## Notas sobre las fuentes de esta bitácora

- El detalle línea por línea de cada sesión (comandos exactos, mensajes de error completos, capturas) queda en el historial de commits de cada repositorio (`git log`) y, para EMA hasta agosto 2026, en los `.docx` originales dentro de `ema-saladillo/docs/`.
- Esta página se actualiza manualmente después de cada sesión de trabajo relevante — no hay automatización todavía que la genere desde los commits.

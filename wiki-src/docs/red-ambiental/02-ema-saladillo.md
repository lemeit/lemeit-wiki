# ![EMA](../assets/logos/ema.svg){: width="36" style="vertical-align:middle;margin-right:8px" } EMAS — app.lemeit.ar/emas

Red meteorológica de estaciones automáticas en Saladillo y 25 de Mayo: temperatura, humedad, presión, viento, lluvia y otros parámetros, comparables entre sí sobre una referencia temporal común.

Repositorio: [github.com/lemeit/lemeit-emas](https://github.com/lemeit/lemeit-emas)

## Las estaciones

| Código | Nombre | Organismo / titular | Equipo | Método de acceso |
|---|---|---|---|---|
| EMA-EET | EEST N°1 "Gral. Savio" | SNIH / INA — Red RMET | Tecmes | API POST JSON (certificado SSL vencido en el servidor → requiere `verify=False`) |
| EMA-CFR | Centro de Formación Rural | CFR Saladillo | Davis Instruments | Scraping HTML (BeautifulSoup) |
| EMA-DC | Defensa Civil — Aeródromo | Municipio Saladillo | Davis / Meteobridge | OCR sobre imagen de cámara (Tesseract + Pillow) — no expone ningún endpoint de datos |
| EMA-CS | Clima Saladillo — B° Falucho | Particular | Davis / Meteotemplate | Endpoint AJAX público (JSON limpio) |
| EMA-25C | 25Clima — 25 de Mayo | N-TecLab / SS Desarrollos (terceros) | Ecowitt-compatible (WU: `IDEMAY14`) | API pública de Weather Underground (PWS) |

## Arquitectura

```
5 scrapers Python (GitHub Actions, cron horario)
        ↓ (API HTTP de Cloudflare)
Cloudflare D1 — tabla unificada "mediciones" (columna "estacion")
        ↓
Worker "ema-saladillo-api" — compatible con el formato PostgREST que usaba Supabase
        ↓
index.html (Cloudflare Pages) — dashboard estático
```

Hasta agosto de 2026 la base era Supabase (PostgreSQL), con 4 tablas separadas. Se migró todo el historial (~30.200 filas) a una tabla D1 unificada; el Worker de compatibilidad expone las mismas rutas y forma de consulta (`select=`, `order=`, `limit=`, `columna=eq.valor`) para no tener que reescribir el dashboard.

## Cómo funcionan los scrapers

Cinco scripts de Python en [`scrapers/`](https://github.com/lemeit/lemeit-emas/tree/main/scrapers), uno por estación — `snih_saladillo_v3.py` (EET), `cfr_saladillo.py`, `dc_saladillo.py`, `cs_saladillo.py` y `wu_25demayo.py` (25C) — cada uno con su propio método de acceso (ver la tabla de arriba) y, por lo tanto, su propio tipo de fragilidad: EMA-DC depende de que el OCR lea bien una imagen; EMA-CFR y EMA-CS dependen de que el HTML/JSON de un sitio de terceros no cambie de formato sin aviso; EMA-25C depende de la disponibilidad de la API de Weather Underground.

Los 5 comparten la misma estructura interna: una función `ciclo()` hace todo el trabajo de una corrida (obtener los datos de la fuente, guardarlos en D1, mostrar un resumen por consola) y devuelve `True`/`False` según si salió bien. Banderas de línea de comandos, comunes a los 5:

- **Sin banderas** — el modo que usa el cron de GitHub Actions: corre `ciclo()` una sola vez y termina con código de salida `1` si algo salió mal (ver "Automatización" más abajo — necesario para que la falla quede registrada).
- **`--loop N`** — modo continuo, corre cada N segundos sin cortar. Pensado para uso manual/supervisado (probar en vivo desde la propia terminal); nunca se usa en GitHub Actions, y a propósito no corta ante una falla puntual (sería raro que un chequeo manual continuo se cierre solo).
- **`--nod1`** — no escribe en D1, para probar el scraping de la fuente sin tocar la base.
- **`--csv`** — además exporta un CSV local.

## Cómo se almacena la información

Todo va a una única tabla `mediciones` en Cloudflare D1 (SQLite), con una columna `estacion` para distinguir la fuente (`EMA-EET` | `EMA-CFR` | `EMA-DC` | `EMA-CS` | `EMA-25C`). Esquema completo en [`d1/schema.sql`](https://github.com/lemeit/lemeit-emas/blob/main/d1/schema.sql): además de las columnas de la medición en sí (`codigo`, `parametro`, `unidad`, `valor`, `valor_texto`, `fecha_hora_utc`), un índice único sobre `(estacion, parametro, fecha_hora_utc)` hace que `INSERT OR IGNORE` descarte solo automáticamente cualquier fila duplicada — útil porque el cron y una corrida manual pueden superponerse, o un scraper puede reintentar.

Los 5 scrapers escriben a través de un helper común, [`scrapers/d1_writer.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/d1_writer.py) — nadie inserta en D1 de forma directa. Dos detalles de ese helper valen la pena conocer, porque los dos salieron de problemas reales:

- **Conversión de hora local a UTC**: acepta tanto años de 4 como de 2 dígitos en la fecha de origen (`DD/MM/YYYY` y `DD/MM/YY`). Esto no es cosmético — la fuente de EMA-CFR cambió de formato de fecha sin aviso en algún momento de septiembre de 2026, y durante casi un mes **todas** sus filas se descartaron en silencio por no matchear el formato esperado (ver bitácora, sesión del 18/09).
- **Red de seguridad ante "cero filas guardadas"**: si hay filas con un valor válido para guardar pero **ninguna** termina insertada (por ejemplo, porque el formato de fecha cambia otra vez y ya no lo reconoce ninguno de los patrones esperados), `guardar_en_d1()` corta con un error en vez de devolver `0` en silencio. Eso es justo lo que le pasó a EMA-CFR: el scraper corría "bien" (sin ninguna excepción) hora tras hora, pero cada fila se descartaba una por una dentro del helper — invisible tanto para el código de salida del scraper como para GitHub Actions. Con esta red, el mismo problema (o uno parecido) ya sale como una falla real y auditable.

## Automatización — GitHub Actions

Workflow: [`.github/workflows/scrapers.yml`](https://github.com/lemeit/lemeit-emas/blob/main/.github/workflows/scrapers.yml).

- **Cron horario** (`5 * * * *`): corre los 5 scrapers, uno por paso, en un único job.
- **`workflow_dispatch`**: permite correr el workflow a mano desde la pestaña Actions de GitHub ("Run workflow"), sin esperar al cron — útil para probar un fix sin tener que esperar hasta la próxima hora en punto.
- **Cron diario** (`5 11 * * *`, 08:05 AR): job aparte, sin relación con EMAS — corre [`scrapers/supabase_ping.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/supabase_ping.py), que hace un ping a un proyecto de Supabase *distinto* del que usaba EMAS, compartido con otras apps personales del autor, para que el plan gratuito no lo pause por inactividad. Quedó en este workflow por comodidad histórica (es el mismo repo donde vivía antes el resto del scraping), pero es independiente de la migración de EMAS a D1: sigue cumpliendo su función mientras esas otras apps sigan corriendo sobre ese Supabase compartido.

**Un detalle importante para saber leer los resultados**: cada paso de scraper tiene `continue-on-error: true` en el workflow — a propósito, para que si una estación falla las otras 4 igual se intenten (sin eso, una falla cortaría el job entero en su primer paso fallido). La consecuencia es que, aunque un scraper individual termine con código de error, **la corrida completa se sigue viendo en verde en la lista general de Actions** — GitHub solo marca ese paso puntual con un triángulo de advertencia si se entra a ver el detalle de esa corrida en particular. Es decir: el código de salida hace que el problema quede *registrado* y sea *auditable*, pero no alcanza para notarlo de un vistazo en la lista de corridas.

Por eso, para saber si las 5 estaciones están sanas de verdad conviene un chequeo activo, no solo mirar si Actions está en verde:

- [`tools/audit_timestamps.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/audit_timestamps.py): script de auditoría de solo lectura (no toca nada del repo ni escribe en D1, consulta la API pública) que compara, por estación, la hora del día en que da el mínimo y el máximo de temperatura contra lo esperado (madrugada / tarde) — si no coincide, es señal de un problema de huso horario o de que la hora que se está guardando no es la real — y hace cuánto no llega un dato nuevo (estación "muda"). Se corre a mano: `python tools/audit_timestamps.py` (o `--dias 14` para una ventana más corta que los 30 días por defecto).
- Abrir manualmente la corrida de Actions que corresponda y expandir cada paso, si se sospecha algo puntual de una estación.

## API pública

Sin autenticación, CORS abierto. Documentación interactiva con ejemplos: [app.lemeit.ar/emas/api.html](https://app.lemeit.ar/emas/api.html).

| Endpoint | Descripción |
|---|---|
| `GET /rest/v1/mediciones_ema` \| `mediciones_cfr` \| `mediciones_dc` \| `mediciones_cs` \| `mediciones_25c` | Mediciones crudas de una estación — `select`, `order`, `limit`, `codigo=eq.X`, `parametro=eq.X`, `horas=N` (ventana relativa), o `desde`/`hasta` (rango de fechas absoluto en UTC) |
| `GET /rest/v1/v_temperatura_comparativa` | Temperatura promedio por hora de las 5 estaciones en columnas paralelas |
| `GET /rest/v1/v_ema_armonizada` | Un mismo parámetro normalizado entre las 5 estaciones (temperatura, humedad, presión, viento, lluvia, etc.) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

Las tres primeras aceptan `&formato=csv`.

`order` solo mira el sufijo `.asc`/`.desc` — el Worker siempre ordena por `fecha_hora_utc` sin importar qué nombre de columna se ponga antes del punto (heredado de cómo estaba armado para Supabase/PostgREST). En la práctica `order=timestamp.desc` y `order=fecha_hora_utc.desc` hacen exactamente lo mismo; no hay forma de ordenar por otra columna.

## Guía de uso de la API

Base URL: `https://api.lemeit.ar/emas` (dominio público compartido con los otros proyectos de la red ambiental, vía Route de Cloudflare — mismo esquema que `api.lemeit.ar/aq`; el `*.workers.dev` original — `ema-saladillo-api...workers.dev` — sigue funcionando igual como respaldo directo, sin el prefijo `/emas`). `app.lemeit.ar/emas` sirve el dashboard estático, no la API (`emas.lemeit.ar` viejo redirige solo ahí) — el propio `index.html` y `api.html` del repo usan esta misma base de API. Las rutas siguen el estilo PostgREST heredado de Supabase: filtros como `columna=eq.valor`, orden con `order=columna.desc`, límite con `limit=N`.

**Últimas 100 mediciones de temperatura de una estación:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100"
```

**Misma consulta pero en CSV, para abrir directo en una planilla:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100&formato=csv" -o temp_eet.csv
```

**Rango de fechas absoluto (UTC) en vez de `horas`:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/mediciones_cfr?parametro=eq.Lluvia&desde=2026-08-01&hasta=2026-08-31&formato=csv" -o lluvia_agosto.csv
```

**Comparar temperatura de las 5 estaciones en paralelo, últimas 48 horas:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/v_temperatura_comparativa?horas=48"
```

**Un parámetro cualquiera armonizado entre las 5 estaciones:**

```bash
curl "https://api.lemeit.ar/emas/rest/v1/v_ema_armonizada?parametro=eq.Humedad&horas=24"
```

**Desde Python + pandas:**

```python
import pandas as pd
df = pd.read_csv("https://api.lemeit.ar/emas/rest/v1/v_temperatura_comparativa?horas=720&formato=csv")
```

Si una consulta devuelve un CSV vacío, probablemente no es un error: puede que esa estación no tenga datos en la ventana pedida (por ejemplo, un corte de transmisión). Conviene probar primero sin `formato=csv` o con una ventana más amplia (`horas=720`) para confirmar si hay datos antes de asumir un problema.

## EMA-25C — la quinta estación, en 25 de Mayo

En agosto/septiembre de 2026 se sumó una quinta estación, en el partido de 25 de Mayo — la primera de la red fuera de Saladillo, en línea con la expansión de [Monitoreo Ambiental Escolar](01-aire-saladillo.md#roadmap) a ese mismo partido. A diferencia de las otras 4, **EMA-25C no es una estación propia del proyecto**: es la estación pública 25Clima (`25clima.ar`), operada por un tercero (N-TecLab / SS Desarrollos) y registrada en la red de Weather Underground como `IDEMAY14`. Se consulta vía la API pública de Weather Underground — no hace falta acceso directo del operador, cualquiera con su propia API key de WU puede leer estaciones públicas ajenas (ver [`scrapers/wu_25demayo.py`](https://github.com/lemeit/lemeit-emas/blob/main/scrapers/wu_25demayo.py) en el repo). Por ser una estación de terceros, está en incorporación: pendiente contactar al operador para confirmar su carácter permanente en el dashboard público. Por su distancia al resto de la red (~80 km), tampoco participa de la interpolación espacial (mapa de calor) del dashboard.

Con esta expansión regional, el proyecto dejó de llamarse "EMA Saladillo" y pasó a **EMAS**. La meta de fondo, igual que en Monitoreo Ambiental Escolar, es habilitar reportes combinados (EMA + AQ) con análisis espacial — hoy limitado porque las estaciones EMA y los sensores de aire no están co-ubicados.

## Hitos técnicos

- **OCR de Defensa Civil**: el sitio no expone ningún endpoint de datos — todo está superpuesto como texto sobre una imagen JPG. Se resolvió con extracción de píxeles blancos (en vez de escala de grises directa, que pierde el texto blanco sobre fondos de color) + Tesseract, con validación en dos capas (rango físico plausible + delta temporal) antes de insertar.
- **Armonización temporal**: las 4 estaciones transmiten con frecuencias distintas; la vista `v_ema_armonizada` normaliza por hora usando una función `IMMUTABLE` (`hora_ar()`), necesaria porque `date_trunc` no es `IMMUTABLE` en PostgreSQL.
- **Migración a GitHub Actions (marzo 2026)**: el sistema original dependía del Programador de Tareas de Windows en una PC física — si se apagaba, se perdían datos. Ver la bitácora para el detalle de la migración y de por qué las tareas de Windows fallaban en ese contexto (`PATH` sin la instalación de Python del usuario).
- **Migración a Cloudflare D1 (agosto 2026)**: como parte de la armonización de los tres portales sobre una misma infraestructura. De 30.213 filas exportadas de Supabase, 9 se descartaron por un timestamp corrupto (error de OCR histórico).
- **API pública (agosto 2026)**: mismo criterio que en Monitoreo Ambiental Escolar — las rutas existentes se documentaron y se les agregó rango de fechas absoluto y export CSV.
- **Quinta estación vía Weather Underground (septiembre 2026)**: para sumar EMA-25C hizo falta una API key propia de Weather Underground, que solo se genera si la cuenta tiene al menos un dispositivo "activo" (con datos reales recientes) — sin tener una estación física propia, se resolvió activando un dispositivo placeholder con datos reales de un sensor PurpleAir ya existente en la red de Monitoreo Ambiental Escolar (ver [`tools/subir_a_wu.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/subir_a_wu.py)). Ver la bitácora para el detalle completo.
- **Color de marca propio y dominio único `app.lemeit.ar` (septiembre 2026)**: el favicon de EMAS pasa de un azul suelto a un celeste de la misma familia tonal que el verde de Monitoreo Ambiental Escolar (`#0097A7`). Junto con esto, los tres portales de la red pasan a convivir bajo un solo dominio (`app.lemeit.ar/aq`, `/emas`, `/wq` — ver [arquitectura de la Red](index.md)); `emas.lemeit.ar` sigue funcionando, redirige solo. El selector de estaciones del header, que en pantallas chicas tapaba el resto de los controles (estado de conexión, switcher de portales, tema), pasa a su propia fila con scroll horizontal.
- **API bajo `api.lemeit.ar/emas` (septiembre 2026)**: `emas.lemeit.ar/rest/v1/*` nunca sirvió datos en los hechos — el dominio apuntaba al Pages estático, que no puede tocar la base D1; la API real siempre vivió solo en el `*.workers.dev` (así la llama el propio `index.html`, vía `SUPA_URL`). Se suma una Route de Cloudflare al mismo Worker (`api.lemeit.ar/emas/*`) para que tenga una URL pública propia y prolija, igual que ya tiene Monitoreo Ambiental Escolar en `api.lemeit.ar/aq`. El `*.workers.dev` original sigue andando igual de respaldo.
- **Auditoría de salud de las 5 estaciones y fin de la pérdida silenciosa de datos de EMA-CFR (septiembre 2026)**: se detectó que los 5 scrapers podían fallar (excepción real, o guardar 0 filas en D1) sin que ninguna corrida de GitHub Actions se viera en rojo — `continue-on-error: true` más un código de salida que siempre era 0 dejaba el problema completamente invisible. Se corrigió en dos capas: cada scraper ahora termina con código de salida distinto de 0 si su ciclo falla, y `scrapers/d1_writer.py` corta con error si hay filas para guardar pero ninguna se llega a insertar (antes devolvía `0` en silencio). Esta segunda capa fue la que encontró el problema real: EMA-CFR venía perdiendo el 100% de sus datos desde hacía casi un mes porque la fuente cambió el formato de fecha (de año de 4 a 2 dígitos) sin aviso, y cada fila se descartaba una por una dentro del helper. Se sumó además [`tools/audit_timestamps.py`](https://github.com/lemeit/lemeit-emas/blob/main/tools/audit_timestamps.py) como chequeo activo de las 5 estaciones (ciclo diurno + frescura de los datos), porque ni siquiera con los exit codes corregidos una corrida de Actions se ve en rojo a simple vista (ver "Automatización" más arriba). Ver la bitácora para el detalle completo, incluyendo la investigación del bug de renderizado de "Rosa de vientos" que salió de la misma sesión.

Ver la [Bitácora del proyecto](99-bitacora.md) para el historial completo, incluyendo el análisis microclimático (efecto isla de calor urbano en EMA-CS) hecho con los primeros datos de las 4 estaciones.

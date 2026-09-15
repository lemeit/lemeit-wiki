# ![EMA](../assets/logos/ema.svg){: width="36" style="vertical-align:middle;margin-right:8px" } EMAS — emas.lemeit.ar

Red meteorológica de estaciones automáticas en Saladillo y 25 de Mayo: temperatura, humedad, presión, viento, lluvia y otros parámetros, comparables entre sí sobre una referencia temporal común.

Repositorio: [github.com/lemeit/emas](https://github.com/lemeit/emas)

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

## API pública

Sin autenticación, CORS abierto. Documentación interactiva con ejemplos: [emas.lemeit.ar/api.html](https://emas.lemeit.ar/api.html).

| Endpoint | Descripción |
|---|---|
| `GET /rest/v1/mediciones_ema` \| `mediciones_cfr` \| `mediciones_dc` \| `mediciones_cs` \| `mediciones_25c` | Mediciones crudas de una estación — `select`, `order`, `limit`, `codigo=eq.X`, `parametro=eq.X`, `horas=N` (ventana relativa), o `desde`/`hasta` (rango de fechas absoluto en UTC) |
| `GET /rest/v1/v_temperatura_comparativa` | Temperatura promedio por hora de las 5 estaciones en columnas paralelas |
| `GET /rest/v1/v_ema_armonizada` | Un mismo parámetro normalizado entre las 5 estaciones (temperatura, humedad, presión, viento, lluvia, etc.) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

Las tres primeras aceptan `&formato=csv`.

## Guía de uso de la API

Base URL: `https://emas.lemeit.ar`. Las rutas siguen el estilo PostgREST heredado de Supabase: filtros como `columna=eq.valor`, orden con `order=columna.desc`, límite con `limit=N`.

**Últimas 100 mediciones de temperatura de una estación:**

```bash
curl "https://emas.lemeit.ar/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100"
```

**Misma consulta pero en CSV, para abrir directo en una planilla:**

```bash
curl "https://emas.lemeit.ar/rest/v1/mediciones_ema?parametro=eq.Temperatura&order=timestamp.desc&limit=100&formato=csv" -o temp_eet.csv
```

**Rango de fechas absoluto (UTC) en vez de `horas`:**

```bash
curl "https://emas.lemeit.ar/rest/v1/mediciones_cfr?parametro=eq.Lluvia&desde=2026-08-01&hasta=2026-08-31&formato=csv" -o lluvia_agosto.csv
```

**Comparar temperatura de las 5 estaciones en paralelo, últimas 48 horas:**

```bash
curl "https://emas.lemeit.ar/rest/v1/v_temperatura_comparativa?horas=48"
```

**Un parámetro cualquiera armonizado entre las 5 estaciones:**

```bash
curl "https://emas.lemeit.ar/rest/v1/v_ema_armonizada?parametro=eq.Humedad&horas=24"
```

**Desde Python + pandas:**

```python
import pandas as pd
df = pd.read_csv("https://emas.lemeit.ar/rest/v1/v_temperatura_comparativa?horas=720&formato=csv")
```

Si una consulta devuelve un CSV vacío, probablemente no es un error: puede que esa estación no tenga datos en la ventana pedida (por ejemplo, un corte de transmisión). Conviene probar primero sin `formato=csv` o con una ventana más amplia (`horas=720`) para confirmar si hay datos antes de asumir un problema.

## EMA-25C — la quinta estación, en 25 de Mayo

En agosto/septiembre de 2026 se sumó una quinta estación, en el partido de 25 de Mayo — la primera de la red fuera de Saladillo, en línea con la expansión de [Monitoreo Ambiental Escolar](01-aire-saladillo.md#roadmap) a ese mismo partido. A diferencia de las otras 4, **EMA-25C no es una estación propia del proyecto**: es la estación pública 25Clima (`25clima.ar`), operada por un tercero (N-TecLab / SS Desarrollos) y registrada en la red de Weather Underground como `IDEMAY14`. Se consulta vía la API pública de Weather Underground — no hace falta acceso directo del operador, cualquiera con su propia API key de WU puede leer estaciones públicas ajenas (ver [`scrapers/wu_25demayo.py`](https://github.com/lemeit/emas/blob/main/scrapers/wu_25demayo.py) en el repo). Por ser una estación de terceros, está en incorporación: pendiente contactar al operador para confirmar su carácter permanente en el dashboard público. Por su distancia al resto de la red (~80 km), tampoco participa de la interpolación espacial (mapa de calor) del dashboard.

Con esta expansión regional, el proyecto dejó de llamarse "EMA Saladillo" y pasó a **EMAS**, manteniendo el dominio `emas.lemeit.ar`. La meta de fondo, igual que en Monitoreo Ambiental Escolar, es habilitar reportes combinados (EMA + AQ) con análisis espacial — hoy limitado porque las estaciones EMA y los sensores de aire no están co-ubicados.

## Hitos técnicos

- **OCR de Defensa Civil**: el sitio no expone ningún endpoint de datos — todo está superpuesto como texto sobre una imagen JPG. Se resolvió con extracción de píxeles blancos (en vez de escala de grises directa, que pierde el texto blanco sobre fondos de color) + Tesseract, con validación en dos capas (rango físico plausible + delta temporal) antes de insertar.
- **Armonización temporal**: las 4 estaciones transmiten con frecuencias distintas; la vista `v_ema_armonizada` normaliza por hora usando una función `IMMUTABLE` (`hora_ar()`), necesaria porque `date_trunc` no es `IMMUTABLE` en PostgreSQL.
- **Migración a GitHub Actions (marzo 2026)**: el sistema original dependía del Programador de Tareas de Windows en una PC física — si se apagaba, se perdían datos. Ver la bitácora para el detalle de la migración y de por qué las tareas de Windows fallaban en ese contexto (`PATH` sin la instalación de Python del usuario).
- **Migración a Cloudflare D1 (agosto 2026)**: como parte de la armonización de los tres portales sobre una misma infraestructura. De 30.213 filas exportadas de Supabase, 9 se descartaron por un timestamp corrupto (error de OCR histórico).
- **API pública (agosto 2026)**: mismo criterio que en Monitoreo Ambiental Escolar — las rutas existentes se documentaron y se les agregó rango de fechas absoluto y export CSV.
- **Quinta estación vía Weather Underground (septiembre 2026)**: para sumar EMA-25C hizo falta una API key propia de Weather Underground, que solo se genera si la cuenta tiene al menos un dispositivo "activo" (con datos reales recientes) — sin tener una estación física propia, se resolvió activando un dispositivo placeholder con datos reales de un sensor PurpleAir ya existente en la red de Monitoreo Ambiental Escolar (ver [`tools/subir_a_wu.py`](https://github.com/lemeit/emas/blob/main/tools/subir_a_wu.py)). Ver la bitácora para el detalle completo.

Ver la [Bitácora del proyecto](99-bitacora.md) para el historial completo, incluyendo el análisis microclimático (efecto isla de calor urbano en EMA-CS) hecho con los primeros datos de las 4 estaciones.

# EMA Saladillo — emas.lemeit.ar

Red meteorológica de 4 estaciones automáticas de Saladillo: temperatura, humedad, presión, viento, lluvia y otros parámetros, comparables entre sí sobre una referencia temporal común.

Repositorio: [github.com/lemeit/ema-saladillo](https://github.com/lemeit/ema-saladillo)

## Las 4 estaciones

| Código | Nombre | Organismo / titular | Equipo | Método de acceso |
|---|---|---|---|---|
| EMA-EET | EEST N°1 "Gral. Savio" | SNIH / INA — Red RMET | Tecmes | API POST JSON (certificado SSL vencido en el servidor → requiere `verify=False`) |
| EMA-CFR | Centro de Formación Rural | CFR Saladillo | Davis Instruments | Scraping HTML (BeautifulSoup) |
| EMA-DC | Defensa Civil — Aeródromo | Municipio Saladillo | Davis / Meteobridge | OCR sobre imagen de cámara (Tesseract + Pillow) — no expone ningún endpoint de datos |
| EMA-CS | Clima Saladillo — B° Falucho | Particular | Davis / Meteotemplate | Endpoint AJAX público (JSON limpio) |

## Arquitectura

```
4 scrapers Python (GitHub Actions, cron horario)
        ↓ (API HTTP de Cloudflare)
Cloudflare D1 — tabla unificada "mediciones" (columna "estacion")
        ↓
Worker "ema-saladillo-api" — compatible con el formato PostgREST que usaba Supabase
        ↓
index.html (Cloudflare Pages) — dashboard estático
```

Hasta agosto de 2026 la base era Supabase (PostgreSQL), con 4 tablas separadas. Se migró todo el historial (~30.200 filas) a una tabla D1 unificada; el Worker de compatibilidad expone las mismas rutas y forma de consulta (`select=`, `order=`, `limit=`, `columna=eq.valor`) para no tener que reescribir el dashboard.

## API pública

Sin autenticación, CORS abierto. Documentación con ejemplos: [emas.lemeit.ar/api.html](https://emas.lemeit.ar/api.html).

| Endpoint | Descripción |
|---|---|
| `GET /rest/v1/mediciones_ema` \| `mediciones_cfr` \| `mediciones_dc` \| `mediciones_cs` | Mediciones crudas de una estación — `select`, `order`, `limit`, `codigo=eq.X`, `parametro=eq.X`, `horas=N` (ventana relativa), o `desde`/`hasta` (rango de fechas absoluto en UTC) |
| `GET /rest/v1/v_temperatura_comparativa` | Temperatura promedio por hora de las 4 estaciones en columnas paralelas |
| `GET /rest/v1/v_ema_armonizada` | Un mismo parámetro normalizado entre las 4 estaciones (temperatura, humedad, presión, viento, lluvia, etc.) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

Las tres primeras aceptan `&formato=csv`.

## Hitos técnicos

- **OCR de Defensa Civil**: el sitio no expone ningún endpoint de datos — todo está superpuesto como texto sobre una imagen JPG. Se resolvió con extracción de píxeles blancos (en vez de escala de grises directa, que pierde el texto blanco sobre fondos de color) + Tesseract, con validación en dos capas (rango físico plausible + delta temporal) antes de insertar.
- **Armonización temporal**: las 4 estaciones transmiten con frecuencias distintas; la vista `v_ema_armonizada` normaliza por hora usando una función `IMMUTABLE` (`hora_ar()`), necesaria porque `date_trunc` no es `IMMUTABLE` en PostgreSQL.
- **Migración a GitHub Actions (marzo 2026)**: el sistema original dependía del Programador de Tareas de Windows en una PC física — si se apagaba, se perdían datos. Ver la bitácora para el detalle de la migración y de por qué las tareas de Windows fallaban en ese contexto (`PATH` sin la instalación de Python del usuario).
- **Migración a Cloudflare D1 (agosto 2026)**: como parte de la armonización de los tres portales sobre una misma infraestructura. De 30.213 filas exportadas de Supabase, 9 se descartaron por un timestamp corrupto (error de OCR histórico).
- **API pública (agosto 2026)**: mismo criterio que en Aire Saladillo — las rutas existentes se documentaron y se les agregó rango de fechas absoluto y export CSV.

Ver la [Bitácora del proyecto](99-Bitacora.md) para el historial completo, incluyendo el análisis microclimático (efecto isla de calor urbano en EMA-CS) hecho con los primeros datos de las 4 estaciones.

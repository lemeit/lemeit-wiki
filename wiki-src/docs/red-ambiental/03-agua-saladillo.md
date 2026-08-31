# Calidad del Agua — wq.lemeit.ar

Monitoreo de calidad de agua de red en Saladillo: arsénico, nitratos, nitritos, fluoruro, metales pesados y parámetros bacteriológicos (coliformes totales, *E. coli*, *Pseudomona aeruginosa*) sobre decenas de puntos de la red municipal (bombas, escuelas, jardines, domicilios).

Repositorio: [github.com/lemeit/agua-saladillo](https://github.com/lemeit/agua-saladillo)

## Origen y estado (agosto 2026)

Existía como archivo suelto (`docs/agua_saladillo.html`) dentro del repo de `ema-saladillo`; se migró a repo propio en agosto de 2026 para evolucionar de forma independiente, como los otros dos proyectos hermanos.

A diferencia de Aire y EMA, **todavía no tiene una API pública de datos**: las muestras (`RAW`) y los límites normativos (`LIM`) siguen embebidos como objeto JS dentro del propio `index.html`, sin base de datos ni backend propio para esa parte — es el ítem principal del roadmap pendiente. Lo que sí tiene backend propio son las **coordenadas de los puntos de muestreo**, servidas desde Cloudflare D1 vía un Worker (mismo patrón que los otros dos portales).

## Origen de los datos

Los valores salen de los protocolos de ensayo que la Municipalidad de Saladillo publica como PDF sueltos en [saladillo.gob.ar/servicios_sanitarios](https://www.saladillo.gob.ar/servicios_sanitarios) — sin tabla, índice ni nombres de archivo consistentes. La carga inicial (87 muestras) fue manual, protocolo por protocolo. Desde agosto de 2026 hay un GitHub Action (`protocolos-ingest.yml`, disparado a mano) que descarga los PDF nuevos y usa la API de Gemini para extraer JSON estructurado de cada uno — el resultado se vuelca a un CSV de staging (`extraidos_pendientes.csv`) para revisión humana antes de mergear al dashboard, nunca directo.

## API existente (parcial)

| Endpoint | Descripción |
|---|---|
| `GET /api/coords` | Coordenadas de los puntos de muestreo — pública, de solo lectura |
| `POST /api/coords` | Editar coordenadas — protegido con header `X-Admin-Key` (401 sin la clave correcta) |
| `GET /tiles/:style/:z/:x/:y{@2x}.png` | Proxy de tiles del mapa hacia CARTO Basemaps |

### Ejemplo de uso

```bash
curl "https://wq.lemeit.ar/api/coords"
```

Las muestras (`RAW`) y los límites normativos (`LIM`) no tienen API todavía — hoy la única forma de consumirlos es leyendo el objeto JS embebido en el `index.html` del dashboard, no hay un endpoint separado para pedirlos.

## Nota sobre Arsénico

Es el único parámetro con una discrepancia real entre normativas: el Código Alimentario Argentino fija 0,01 mg/L (adoptado de la OMS) mientras que el texto vigente de la Ley PBA 11.820 (Anexo A) todavía dice 0,05 mg/L, sin actualizar — aunque en la práctica la Provincia adhiere al valor de OMS/CAA, que es el que citan los propios protocolos municipales. El dashboard muestra **ambos** límites en vez de elegir uno, para dejar ver la brecha entre la norma escrita y la práctica real.

## Roadmap pendiente

- Backend propio para `RAW`/`LIM` (Cloudflare D1 + Worker), siguiendo el mismo patrón que ya usan `ema-saladillo` y `purpleair-saladillo` — el paso que habilitaría una API pública de datos igual que en los otros dos portales, y que la ingesta automática escriba directo a la base en vez de a un CSV de staging.

Ver la [Bitácora del proyecto](99-bitacora.md) para más contexto sobre la armonización de los tres portales.

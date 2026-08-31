# lemeit.ar — Documentación

Wiki madre de los proyectos de **lemeit.ar**: documentación técnica, guías de uso y bitácora de desarrollo, todo en un mismo lugar. Cada proyecto tiene su propia sección en el menú; esta portada es solo el punto de entrada.

## Proyectos documentados

| Proyecto | Qué es | Sitio | Sección |
|---|---|---|---|
| 🌎 Red Ambiental Saladillo | Tres portales hermanos de monitoreo ambiental (aire, meteorología, agua) sobre Cloudflare | [aq](https://aq.lemeit.ar) · [emas](https://emas.lemeit.ar) · [wq](https://wq.lemeit.ar) | [Ver sección](red-ambiental/index.md) |
| 🎓 Eureka Tutor | Tutor socrático de ciencias asistido por IA, para secundario y curso de ingreso universitario | [tutor.lemeit.ar](https://tutor.lemeit.ar) | [Ver sección](eureka-tutor/index.md) |

Nuevos proyectos se van sumando como secciones nuevas a medida que se documentan — esta tabla y el menú lateral se actualizan cada vez.

## Sobre esta wiki

- Generada con [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/), el mismo stack que usa la wiki de [DVBA](https://github.com/lemeit/DVBA) (`wiki-src/` → build automático vía GitHub Actions → `site/` → GitHub Pages).
- El código fuente de cada página vive en `wiki-src/docs/` del repo [`lemeit/lemeit-wiki`](https://github.com/lemeit/lemeit-wiki), organizado en una carpeta por proyecto.
- Todo el contenido es público: no hay nada que ocultar en la parte técnica ni de desarrollo. La idea es que cualquiera —un colega, un alumno, alguien de la Municipalidad, un futuro yo mismo dentro de un año— pueda entender qué hace cada proyecto, con qué tecnología está armado y cómo se usa, sin tener que pedir explicaciones por privado.
- Se actualiza a mano después de cada sesión de trabajo relevante en cualquiera de los proyectos — no hay automatización todavía que la genere sola desde los commits.

## Cómo agregar un proyecto nuevo a esta wiki

1. Crear una carpeta nueva en `wiki-src/docs/<nombre-proyecto>/` con al menos un `index.md`.
2. Sumar la sección al `nav:` de `wiki-src/mkdocs.yml`.
3. Agregar la fila correspondiente a la tabla de arriba.
4. Commitear y pushear a `main` — el build y el deploy a `wiki.lemeit.ar` son automáticos.

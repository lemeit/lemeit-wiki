# Mantenimiento y publicación

Actualizado el 6 de noviembre de 2026.

## Fuentes independientes

- Aplicación: repositorio privado `lemeit/lemeit-tex`, archivos servidos desde `site/`.
- Documentación pública: `lemeit/lemeit-wiki`, fuentes en `wiki-src/docs/tex/`.
- Profe Lamaita: `lemeit/aboutme`; su antigua carpeta `apps/tex` no es la fuente de publicación de TeX.

La aplicación no usa Hugo ni Quartz. Los PDF de las notas de Profe, generados
por su propio flujo, son distintos de los documentos compilados por usuarios de TeX.

## Compilación en el navegador

HTML y CSS organizan la interfaz; JavaScript controla proyectos y edición.
El modo preciso utiliza **WasmTex/pdfTeX, TeX Live 2025**, compilado a WebAssembly
y ejecutado en un Web Worker del navegador. PDF.js presenta el PDF y su capa de
texto. SyncTeX permite relacionar posiciones con archivos y líneas reales.

Desactivar SyncTeX conserva el motor compatible **SwiftLaTeX/pdfTeX 2020**, sin
esa correspondencia. Los recursos y las cachés de ambos motores están separados.
Motor, formato y espejo del modo preciso deben mantenerse como un conjunto
compatible; no actualizar solo una de esas piezas.

Cloudflare entrega la aplicación y un puente limitado a consultas GET/HEAD de
archivos públicos de un espejo fijo de TeX Live. No recibe documentos para
compilarlos, no acepta destinos arbitrarios y no almacena proyectos de alumnos.
El trabajo se guarda en el navegador mediante IndexedDB; no hay colaboración,
cuentas ni sincronización automática de documentos.

## De GitHub al sitio público

| Paso | Sistema | Qué sucede |
| --- | --- | --- |
| PR hacia `main` | GitHub Actions | Instala dependencias y ejecuta `npm run deploy:check` |
| Fusión autorizada | GitHub | Actualiza el código de `main` |
| Build de producción | Cloudflare Workers Builds | Toma `main` y ejecuta `npm run deploy` |
| Sitio público | Worker `lemeit-tex` | Sirve `tex.lemeit.ar` |

Workers Builds usa directorio raíz `/`, sin comando de build separado; el comando
de despliegue es `npm run deploy`, que vuelve a validar antes de publicar.
GitHub Actions no despliega por sí mismo; la integración de Cloudflare sí lo hace
al actualizar `main`. **Fusionar o subir a `main` puede publicar**, por lo que se
requiere autorización. Comprobar el resultado del build y el sitio público.

Los tokens se administran en Cloudflare. No incluir secretos en Git, documentación
ni registros. `npm run deploy` también existe como alternativa manual autorizada.

## Verificaciones y caché

En el repositorio de TeX:

```sh
npm ci
npm run validate
npm run deploy:check
```

`deploy:check` comprueba el paquete sin desplegar. Además, verificar en navegador
los flujos afectados, la identidad y los tamaños de pantalla pertinentes.

Cuando cambia la interfaz, renovar `SHELL` en `site/sw.js`, sus expectativas en
`tests/offline.test.cjs` y las URLs versionadas de los recursos modificados.
Esto evita mezclar versiones. No borrar proyectos ni cachés de paquetes para
renovar la apariencia. No reconstruir logos o guías por rutina.

La wiki se valida separadamente:

```sh
cd wiki-src
mkdocs build --strict --site-dir ../site-check
```

No commitear la salida generada. Cloudflare Pages compila y publica la wiki desde
su propio `main`; modificar la wiki no cambia la aplicación.

## Registro de la actualización

| PR de `lemeit-tex` | Cambio |
| --- | --- |
| #1 | Motor preciso, SyncTeX y menús sobre paneles flotantes |
| #2 | Tres vistas, asistencia de edición y galería de apariencia |
| #3 | Zoom, ajuste al ancho y página completa |
| #4 | Búsqueda sobre separadores, barras y paneles |

Estado publicado tras #4: `main` en `d610019`, caché de interfaz
`lemeit-tex-shell-v64-busqueda`, 71 pruebas y empaquetado comprobados.
Los nombres y diseños alternativos se consideran propuestas mientras no haya una
decisión explícita: el producto publicado continúa siendo **TeX**.

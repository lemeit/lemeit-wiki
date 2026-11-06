# TeX · Documentos en LaTeX

[![sitio: tex.lemeit.ar](https://img.shields.io/badge/sitio-tex.lemeit.ar-009688?style=flat-square&labelColor=555555)](https://tex.lemeit.ar/)

TeX es el editor de documentos LaTeX de Lemeit. Ofrece edición, vista previa de PDF, modelos de inicio y proyectos locales en una aplicación web con interfaz en español. Estado: **en desarrollo**. Documentación actualizada el 6 de noviembre de 2026.

## Accesos

- [Abrir el editor](https://tex.lemeit.ar/).
- [Guía PDF](https://tex.lemeit.ar/guia-tex.pdf) y [proyecto editable de la guía](https://tex.lemeit.ar/guia-tex.zip).
- [Ficha del proyecto en Profe Lamaita](https://profe.lemeit.ar/projects/tex/).
- [Repositorio independiente lemeit-tex (acceso restringido)](https://github.com/lemeit/lemeit-tex).

## Funciones actuales

- Vistas Código, Dividida y Vista previa, con tamaños ajustables.
- Asistencia local: colores de sintaxis, sugerencias con Tab, cierre de llaves y sangría.
- Menús Archivo, Edición, Insertar, Formato, Vista y Ayuda, con acceso por teclado.
- Buscar/reemplazar, ir a línea, comentar y localizar errores.
- SyncTeX: seleccionar texto o Ctrl/Comando + clic en el PDF para ir al código; Alt + F9 para volver al PDF.
- Zoom con porcentaje, Ancho y Página; esquemas visuales independientes para editor, consola y PDF.
- Ayuda y consola flotantes o acoplables, redimensionables.

La interfaz, las plantillas y la guía de la aplicación siguen en español. La versión inglesa de la aplicación está pendiente; esta wiki sí ofrece documentación en inglés.

Ver [uso, plantillas y encabezados](uso-y-plantillas.md) y [mantenimiento y publicación](mantenimiento.md).

## Flujo de trabajo

1. Elegir un modelo, comenzar en blanco o abrir un proyecto compatible.
2. Editar el documento y revisar el PDF; la compilación puede ser manual o automática.
3. Consultar Ayuda y la consola si hay errores de LaTeX.
4. Descargar el PDF y exportar una copia del proyecto para conservar las fuentes.

El editor admite proyectos con varios archivos y selección del documento principal. Archivo → Abrir proyecto ZIP importa textos LaTeX, bibliografía y figuras compatibles, conservando su organización. El PDF es el documento final; el ZIP y la copia `.tex.json` permiten continuar la edición.

## Almacenamiento y respaldo

Los proyectos y sus recursos se guardan en IndexedDB del navegador. No hay sincronización automática entre computadoras ni una cuenta de almacenamiento en la nube. Borrar los datos del navegador puede eliminar los proyectos: exportar ZIP o copia TeX es el respaldo portátil.

## Trabajo sin conexión

La primera visita necesita internet. Edición → Preferencias → Uso sin conexión permite acceder a las opciones. «Preparar sin conexión» guarda la aplicación y los recursos utilizados por el proyecto preparado. «Comprobar recursos guardados» prueba la compilación sin consultar el espejo de paquetes. Un paquete no descargado requiere conexión; el navegador también puede liberar su almacenamiento.

## Arquitectura y límites

- Sitio estático independiente de Hugo y Quartz, con motor LaTeX ejecutado en el navegador.
- Publicación de la aplicación mediante Cloudflare Workers Static Assets; esta wiki se publica por su propio flujo.
- Dependencias y licencias registradas en `site/vendor/NOTICE.md`; no se atribuye una licencia única a todos los componentes.
- No ejecuta scripts, Biber, XeLaTeX ni programas externos. No se garantiza compatibilidad con cualquier proyecto LaTeX.
- Importación ZIP: hasta 42 MB, 300 archivos admitidos y 50 MB descomprimidos; hasta 10 MB por archivo y 1 MB por texto, según el desarrollo documentado.

## Identidad

La marca geométrica TeX usa naranja `#9B3D00` y verde azulado `#009688`. Es una herramienta de Lemeit, enlazada desde Proyectos Lemeit. Su símbolo es distinto de la firma manuscrita personal de Profe Lamaita.

## Código, pruebas y publicación

La aplicación se mantiene en `lemeit/lemeit-tex`, separada de `aboutme`.

- **GitHub** guarda el código y ejecuta las validaciones y la comprobación del empaquetado mediante Actions.
- **Cloudflare** sirve `site/` en `tex.lemeit.ar`, con el recurso existente `lemeit-tex`.
- **El navegador** ejecuta WasmTex/pdfTeX 2025 por defecto (SyncTeX), o SwiftLaTeX en modo compatible, mediante WebAssembly; PDF.js presenta el resultado.
- **Un Worker restringido** facilita únicamente archivos públicos de un espejo fijo de TeX Live. No recibe documentos para compilarlos.

GitHub Actions valida y no despliega por sí mismo. **Cloudflare Workers Builds está conectado al repositorio y a `main`**: una fusión o actualización de esa rama inicia la publicación con `npm run deploy`. El responsable debe autorizarla. El build vuelve a ejecutar las pruebas antes de desplegar. Ver [el flujo de publicación](mantenimiento.md).

Al 6 de noviembre de 2026, las mejoras de vistas, edición asistida, apariencia, zoom y búsqueda están integradas en `main` mediante los PR #1 a #4. Se verificaron 71 pruebas y el empaquetado; caché de interfaz `lemeit-tex-shell-v64-busqueda`.

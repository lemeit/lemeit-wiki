# Uso, plantillas y encabezados

Actualizado el 6 de noviembre de 2026. Interfaz actual en español.

## Empezar y conservar el trabajo

En **Archivo → Elegir plantilla** podés comenzar desde cero o elegir un modelo
guiado de informe, trabajo práctico o monografía. El diseño del portal ofrece
**Apunte en columnas** y **Clase y actividades**. Las opciones de portada, color,
tipografía, columnas y encabezado se aplican según el modelo; una plantilla
editorial conserva sus propias reglas.

El nombre junto a los menús determina el nombre de las descargas. Cambiarlo no
modifica el título impreso, que depende de `\title` o `\LemeitTitulo`.

| Descarga | Qué conserva |
| --- | --- |
| PDF | Resultado para leer, imprimir o entregar |
| Proyecto ZIP | Código, imágenes y demás archivos del proyecto |
| Copia TeX (`.tex.json`) | Proyecto y selección de archivos principal y activo |
| Archivo actual | Solo el archivo abierto en el editor |

El guardado automático queda en este navegador. No es un respaldo en una cuenta
ni se sincroniza entre equipos. Exportá un ZIP o copia TeX para seguir editando.
Elegir otra plantilla conserva una copia recuperable del proyecto anterior.

## Edición y lectura

El selector **Código / Dividida / Vista previa** cambia el espacio de trabajo.
Podés arrastrar el separador en vista dividida. El PDF tiene botones **− / +**, un
porcentaje entre 25 y 300 %, **Ancho** y **Página**. El ajuste inicial muestra el
ancho en vista dividida y la página completa en vista previa ampliada; las
elecciones explícitas se recuerdan localmente. El zoom no cambia el PDF descargado.

En **Edición → Preferencias** podés elegir esquemas independientes para el editor,
la consola y la lectura del PDF. Sepia y oscuro también transforman visualmente
las imágenes del PDF; usá Original para revisar sus colores reales.

Las sugerencias aparecen al escribir comandos como `\sec`: flechas para elegir,
**Tab** para insertar y **Esc** para cerrar. Enter mantiene la nueva línea. Se
pueden desactivar el resaltado, las sugerencias, el cierre de llaves y la sangría.
Las sugerencias son un catálogo básico y los colores no validan toda la sintaxis.

| Acción | Atajo |
| --- | --- |
| Buscar / reemplazar | Ctrl o Comando + F / H, dentro del editor |
| Ir a línea | Ctrl + Alt + G |
| Comentar / descomentar | Ctrl o Comando + / |
| Actualizar vista | Ctrl o Comando + Enter |
| Guardar copia del proyecto | Ctrl o Comando + S |
| Ir al error de LaTeX | Alt + F8 |
| Código → PDF | Alt + F9 |
| Ayuda | F1 |
| Activar menús | Alt o F10; flechas y Enter; Esc para volver |

Con SyncTeX activo, seleccionar texto o hacer Ctrl/Comando + clic en el PDF
localiza la línea real de su archivo. Si editaste después de compilar, esperá al
PDF actualizado. La correspondencia puede abarcar un párrafo o un entorno de
ecuación; no es necesariamente carácter por carácter.

**Ayuda** y **Consola** se pueden mover por su cabecera, redimensionar y acoplar
a los bordes. La búsqueda permanece sobre separadores y paneles, sin bloquear la
edición. Esc cierra la búsqueda y devuelve el foco al código.

## Dónde están los modelos y los logos

En el repositorio privado `lemeit/lemeit-tex`:

| Ruta | Función |
| --- | --- |
| `site/templates/` | Archivos de modelos y recursos |
| `site/templates.js`, `site/template-builder.js` | Catálogo y construcción de modelos |
| `site/lemeit-template.js` | Perfiles institucionales y diseño del encabezado |
| `site/templates/lemeit/` | Logos de escuelas, materias y guía |

Los escudos de Madre Teresa y EEST N.º 1 se distribuyen como `madre-teresa.png` y
`eest1.png`; las materias usan `intro-fisica.png` y `lab-industrias.png`. Al crear
el proyecto, los elegidos se copian como `logo-institucion.png` y
`logo-materia.png`. El perfil personalizado no incorpora logos automáticamente.
Las marcas institucionales conservan sus derechos.

Cambiar un documento no modifica el modelo original. Cambiar el modelo en el
repositorio afecta a futuros documentos y requiere publicar la aplicación; no
reescribe los proyectos guardados por los usuarios.

## Editar un encabezado existente

Los textos están al principio del `.tex`, por ejemplo:

```latex
\newcommand{\LemeitInstitucion}{EEST N°1 Gral. Savio}
\newcommand{\LemeitLocalidad}{Saladillo}
\newcommand{\LemeitMateria}{Laboratorio de Industrias}
\newcommand{\LemeitCurso}{7° TQ}
```

El bloque `\LemeitCabecera` contiene la distribución. `minipage` agrupa elementos,
`\hspace` controla espacios horizontales y `\includegraphics` el tamaño del logo.
Para bajar únicamente el logo 2 mm, envolvé su comando:

```latex
\raisebox{-2mm}{%
  \includegraphics[width=9mm,height=12mm,keepaspectratio]{logo-institucion.png}%
}
```

Un valor positivo lo sube; uno negativo lo baja. El ajuste exacto depende del
espacio transparente de la imagen y del bloque de texto. Para dar a la localidad
el mismo tamaño, color y peso que a la institución, usá dentro del bloque de texto:

```latex
\raggedright
\fontsize{9}{10.5}\selectfont
\color{black}
\textbf{\LemeitInstitucion}\\
\textbf{\LemeitLocalidad}
```

Reemplazá las líneas anteriores que aplicaban `\scriptsize` y gris a la localidad;
no las dejes duplicadas. El PDF es la vista del resultado: no permite arrastrar
logos para cambiar el código. Podés insertar otra imagen y referenciar su nombre,
o reemplazar el archivo del ZIP conservando el nombre y volver a importar.

## Plantillas externas y funcionamiento sin conexión

Importá el ZIP completo: copiar solo el `.tex` no trae clases, imágenes ni
bibliografía. El motor admite proyectos pdfLaTeX y BibTeX compatibles; no ejecuta
Biber, scripts ni programas externos. Una plantilla puede requerir otro motor.

En **Edición → Preferencias → Uso sin conexión**, prepará el proyecto y comprobá
los recursos guardados. La comprobación utiliza un motor nuevo y solo la caché.
Una plantilla nueva puede necesitar otros paquetes; no está incluida toda TeX Live
y el navegador puede liberar almacenamiento. Conservá siempre copias exportadas.

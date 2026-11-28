# Manual de uso de ΣIM

Esta es la **guía operativa central** de [ΣIM · Simulación de procesos](https://sim.lemeit.ar/). Se actualiza junto con la aplicación. Para los fundamentos científicos, ejercicios y modelos, consultá los [16 capítulos de la wiki](index.md). Para enseñar con un ejemplo, abrí la biblioteca de **ejemplos** desde el menú *Ejemplos* del simulador.

ΣIM es un editor web para preparar diagramas estacionarios y estudiar la dinámica de tanque mediante DWSIM o un modelo didáctico independiente. **No es la versión de escritorio de DWSIM**, ni incluye todas sus operaciones. Los resultados de cálculo son predicciones de modelos y no equivalen a una validación experimental.

## 1. Entrar al portal

Accedé a [sim.lemeit.ar](https://sim.lemeit.ar/). Los alumnos ingresan con un código vigente de clase de 12 caracteres, sin registrar una cuenta de correo personal. En el pie del editor se informa la clase, la vigencia de la sesión y la opción **Salir**. Guardá el trabajo antes de salir: una sesión puede vencer o ser desactivada.

La administración de clases exige credenciales y permisos específicos: consultar [Acceso y clases](acceso.md). Las operaciones de administración no están disponibles para alumnos.

## 2. Comenzar y abrir proyectos

1. Elegí **Archivo → Nuevo proyecto** y seleccioná los componentes y el paquete de propiedades.
2. También podés abrir un caso desde **Ejemplos** o retomar un trabajo desde **Archivo → Abrir biblioteca en la nube…**.
3. Pulsá **Guardar en nube** en la barra. Si es tu primer guardado, ingresá con tu alias y clave de alumno o creá una identidad dentro de la clase. Los cambios posteriores se guardan automáticamente.
4. Desde **Archivo** y **Gráficas**, exportá las tablas, las imágenes o el PDF que necesites para el informe. Administración puede consultar trabajos por clase y descargar una copia JSON de respaldo desde **Archivo**. El nombre de esa copia se arma con el título del proyecto.

Cuando abrís un proyecto, ΣIM recupera la última corrida DWSIM si se guardó con él. Los proyectos anteriores pueden abrirse y requieren un cálculo si todavía no tienen una corrida guardada. Si cambiás condiciones físicas, los resultados anteriores dejan de representar el estado actual y deben recalcularse.

### Guardar y recuperar proyectos en Desktop

ΣIM Desktop es una aplicación Tauri en desarrollo que usa el motor local. En la versión **0.53.3-alpha.1**, el disquete, **Ctrl + S** y **Archivo → Guardar copia de proyecto (.sim)…** abren el selector del sistema. Confirmá nombre y carpeta. El aviso posterior indica la ruta escrita. Si cancelás, podés seguir trabajando en el proyecto.

La copia `.sim` conserva el diagrama, la configuración de Gráficas y la última corrida válida, cuando existe. **Archivo → Abrir proyecto…** admite `.sim` y los archivos `.json` de Online. **Biblioteca local…** guarda dentro de Desktop en el equipo actual y **Recuperar borrador local** retoma el último borrador.

| Dónde estás trabajando | Acción | Dónde queda |
| --- | --- | --- |
| Online | **Guardar en nube** | En la biblioteca de tu identidad de alumno |
| Online | **Descargar copia JSON** | En las descargas del navegador o en la ubicación elegida |
| Desktop | **Guardar copia de proyecto (.sim)…** | En la carpeta elegida en el selector nativo |
| Desktop | **Biblioteca local…** | En el almacenamiento local de Desktop |

[Guía de ejecución de Desktop](https://github.com/lemeit/lemeit-sim/tree/main/desktop).

## 3. Diagrama, equipos, corrientes y conexiones

La **biblioteca Equipos y corrientes** contiene los símbolos disponibles. Se pueden agregar al diagrama con clic o arrastre, mover los símbolos y conectar los extremos de las corrientes con los puertos del equipo. Cada conexión debe enlazar una corriente y una operación, no dos equipos directamente.

Seleccioná una corriente o un equipo para editar sus datos en **Propiedades**: caudal, temperatura, presión, composición, paquete o parámetros del equipo según corresponda. Los enlaces bajo cada objeto apuntan a los capítulos técnicos de esta wiki.

Un dibujo coherente es condición necesaria, pero no suficiente, para obtener un cálculo físicamente válido. Consultá [Diagramas y conexiones](diagramas-conexiones.md) y [Grados de libertad](grados-libertad-especificaciones.md).

## 4. Organizar el espacio de trabajo

En pantallas de escritorio, **Equipos y corrientes** y **Propiedades** pueden cambiar de lado y abrirse como paneles flotantes dentro de la página. Cuando un panel está flotante, arrastrá su **cabecera** para desplazarlo y utilizá **Acoplar** para regresarlo al editor. La distribución se puede ajustar acoplando los paneles en distintos bordes y cambiando sus tamaños. La distribución se conserva localmente en ese navegador, no dentro del JSON del proceso.

Los bordes laterales permiten ajustar el ancho de los paneles acoplados. **Balances y corrientes** puede separarse en otra ventana sincronizada mediante su icono **↗**, cambiar de altura arrastrando el borde superior, ocultarse y recuperarse desde **Ver**.

El **diagrama** puede ampliarse mediante **Solo diagrama**, separarse en otra ventana sincronizada mediante **Ver → Abrir diagrama en otra ventana** y mostrarse en pantalla completa. Las ventanas separadas requieren que el navegador permita ventanas emergentes. Los paneles flotantes internos no son ventanas independientes del sistema operativo.

En teléfonos o pantallas pequeñas se prioriza la disposición acoplada y adaptable: algunas funciones de flotación se desactivan para evitar superposiciones.

### Ajustes visuales y de navegación

- Arrastrá los separadores verticales para ajustar el ancho y el separador horizontal ↕ para repartir la altura entre paneles apilados en un mismo lateral. El doble clic centra este último; también admite teclado.
- Reordená los grupos de herramientas por arrastre o Alt + flechas, y restablecé su distribución desde Ver.
- El selector claro/oscuro está en la cabecera y se comparte con acceso y administración. Los símbolos de equipos del diagrama son los de DWSIM; las corrientes de energía se distinguen por trazos.
- Los indicadores +/− abren o cierran detalles. Explorar proceso mantiene su tamaño y desplaza internamente ecuaciones y balances sin mover el lienzo.
- En el catálogo de compuestos aparecen favoritos, contador de coincidencias y Mostrar más; la búsqueda abarca el catálogo completo.

## 5. Modelos termodinámicos, composición y unidades

Elegí un modelo apropiado antes de calcular. Por ejemplo, **IAPWS-IF97** es adecuado para agua pura dentro de sus límites; mezclas como agua–etanol pueden requerir modelos de coeficientes de actividad y parámetros aplicables. Consultá [Modelos termodinámicos](modelos-termodinamicos.md).

Desde **Herramientas → Configuración** se puede cambiar la representación de unidades, manteniendo las condiciones físicas internas. La calculadora de unidades y composición ayuda a convertir caudales y fracciones. Distinguir base **másica** de **molar**, y presión **absoluta** de **manométrica**. Las estimaciones de gas ideal y presión parcial no sustituyen un cálculo de equilibrio.

Ver [Unidades y bases](unidades-bases-conversiones.md).

## 6. Comprobar y calcular

1. Completá las condiciones de alimentación y los parámetros de cada operación.
2. Usá **Herramientas → Diagnóstico del proyecto** para revisar especificaciones y conexiones. Corregí advertencias sobre puertos, datos incompletos o límites de operaciones.
3. En el pie, seleccioná explícitamente el motor **Windows** o **Debian** y comprobá su disponibilidad. Con motor fijado no hay conmutación automática. El modo Automático verifica disponibilidad antes de calcular, sin reenviar una solicitud incierta.
4. Pulsá **▶ Calcular** y verificá la luz de estado. La solicitud llega al gateway y luego al motor DWSIM seleccionado.
5. Consultá el panel de **Balances y corrientes**, el modelo, las temperaturas, presiones, caudales y los residuos de cada equipo.
6. Si el cálculo es válido, seleccioná **Explorar proceso…** para recorrer sus resultados, ecuaciones y balances sin volver a ejecutar DWSIM.

Si un motor no está disponible, todavía podés editar y guardar el proyecto, pero **no inventar resultados**. Un cálculo completado tampoco garantiza por sí solo que se eligió un buen modelo físico.

Ver [Diagnóstico y validación](diagnostico-validacion.md).

## 7. Exportar, comparar y documentar

Desde **Archivo → Exportar tablas** descargá corrientes y balances en **CSV** o **Excel**, o prepará el **Reporte PDF · diagrama y balances…** con tamaño A4/A3/Oficio y rótulo. Revisá las unidades de las columnas, porque las descargas pueden utilizar unidades canónicas diferentes de la representación elegida para la pantalla.

Para una práctica reproducible conservá el proyecto en la biblioteca, la fecha, el motor seleccionado, las condiciones, el paquete termodinámico, los resultados y las advertencias. Se recomienda contrastar balances de materia y energía por componente y por equipo. Ver [Reproducibilidad e informes](reproducibilidad-informes.md).

## 8. Ejercicios y ayuda contextual

Los ejemplos sirven como casos de estudio: mezcla, operaciones térmicas, divisor, mezclas multicomponentes, reactores, flash, bomba/válvula e intercambiador. La guía de ejercicios del portal contiene los resultados documentados por caso; se accede desde **Ejemplos** y permite volver al editor sin sustituir el proyecto.

En el diagrama se pueden agregar anotaciones editables; el editor de proyecto reúne actividad, notas y datos de informe. El **Ejemplo 13** incluye una actividad de predicción y comparación de temperaturas que se inicia desde el inspector del intercambiador cuando está cargado el ejemplo. Sus valores históricos se distinguen de los resultados nuevos del motor.

Ruta sugerida para estudiar: [Fundamentos de corrientes](fundamentos-corrientes-balances.md) → [Energía](energia-operaciones-termicas.md) → [Bombas y válvulas](bombas-valvulas-presion.md) → [Flash](equilibrio-fases-flash.md) → [Prácticas integradoras](practicas-integradoras.md).

## 9. Problemas frecuentes

| Situación | Qué revisar |
|---|---|
| No se puede calcular | Motor seleccionado, conexión, sesión válida y diagnóstico de parámetros |
| Faltan resultados después de un cambio | Recalcular el proyecto; se invalidaron los resultados previos |
| No se ve una ventana separada | Permisos de ventanas emergentes en el navegador |
| Se perdió un panel flotante de vista | Recuperarlo desde **Ver → Paneles** o volver a acoplarlo |
| La composición no cierra | Suma de fracciones, base másica/molar y conversiones |
| El motor responde pero el cálculo falla | Datos de equipo, conexiones, paquete de propiedades y validación |

Para problemas de acceso: [Acceso y clases](acceso.md). Para los límites y la interpretación científica: [Diagnóstico y validación](diagnostico-validacion.md).

## 10. Interpretación energética del intercambiador

Un intercambiador puede transferir aproximadamente **40 kW entre sus dos circuitos** y, al mismo tiempo, tener un cambio neto de entalpía global cercano a cero. Este último valor **no significa ausencia de intercambio térmico**. Antes de informar una potencia, distinguí calor cedido por el lado caliente, recibido por el frío y residuo global.

La corrección de etiquetas y columnas está integrada desde 0.47.5. Comprobá la versión efectiva en Ayuda → Acerca de y contrastá los lados mediante entalpías y caudales.

## 11. Dinámica y Gráficas

1. Abrí **Ejemplos → 20**. El proyecto abre en **Dinámico**. Comprobá que el motor tenga capacidad dinámica y revisá duración y paso desde **Configuración dinámica**.
2. Pulsá **Calcular**. DWSIM integra el circuito de agua con tanque y dos válvulas. **Gráficas** abre con el nivel frente al tiempo. El Ejemplo 15 sigue en régimen estacionario.
3. En **Ver → Gráficas** podés volver al panel, cambiar las variables XY y añadir una segunda curva. Si las unidades de Y difieren, la segunda curva usa el eje derecho. Podés ajustar las escalas por separado.
4. Ajustá el subtítulo, la paleta, el color, la representación y el grosor de cada serie. El proyecto guarda el formato, la selección de variables y la última corrida de DWSIM. Al retomarlo, podés volver a ver la curva y las muestras. Exportá SVG, PNG o JPG para un informe y CSV para usar los datos en una planilla.

La comprobación con DWSIM en Windows y Debian registró 121 muestras en 600 s, con paso de 5 s, nivel final de aproximadamente 0,333 m y descarga final de aproximadamente 10 kg/s. El modelo didáctico del tanque tiene su propio cálculo y reproducción temporal. Ante un fallo de DWSIM, el simulador informa el error. Consultá la [guía de Gráficas](https://github.com/lemeit/lemeit-sim/blob/main/docs/graficas-cientificas.md) y el [registro del Ejemplo 20](https://github.com/lemeit/lemeit-sim/blob/main/docs/ejemplo-20-tanque-dinamico-dwsim.md).

## 12. Alcance y fuentes

El cálculo general del PFD es estacionario; la dinámica del tanque es un módulo específico experimental, no un integrador de cualquier diagrama. El gateway incluye un primer soporte de recirculación simple; los Ejemplos 17–19 requieren todavía validación cruzada y pruebas de convergencia antes de considerarse consolidados. Las columnas de destilación, los PFR/CSTR cinéticos y otros equipos avanzados de DWSIM **no deben presentarse como implementados en este editor**.

**Documentación fuente:** [Repositorio ΣIM](https://github.com/lemeit/lemeit-sim) · [DWSIM: tutoriales oficiales](https://dwsim.org/tutorials/en/index.html). Esta guía es documentación propia de ΣIM, no una traducción oficial de DWSIM.

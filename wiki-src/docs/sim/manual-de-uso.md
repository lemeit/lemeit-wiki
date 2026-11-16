# Manual de uso de ΣIM

Esta es la **guía operativa central** de [ΣIM · Simulación de procesos](https://sim.lemeit.ar/). Se actualiza junto con la aplicación. Para los fundamentos científicos, ejercicios y modelos, consultá los [16 capítulos de la wiki](index.md). Para enseñar con un ejemplo, abrí la biblioteca de **13 casos** desde el menú *Ejemplos* del simulador.

ΣIM es un editor web para preparar diagramas estacionarios que se calculan mediante motores DWSIM seleccionados manualmente. **No es la versión de escritorio de DWSIM**, ni incluye todas sus operaciones. Los resultados de cálculo son predicciones de modelos y no equivalen a una validación experimental.

## 1. Entrar al portal

Accedé a [sim.lemeit.ar](https://sim.lemeit.ar/). Los alumnos ingresan con un código vigente de clase de 12 caracteres, sin crear una cuenta personal. En el pie del editor se informa la clase, la vigencia de la sesión y la opción **Salir**. Guardá el trabajo antes de salir: una sesión puede vencer o ser desactivada.

La administración de clases exige credenciales y permisos específicos: consultar [Acceso y clases](acceso.md). Las operaciones de administración no están disponibles para alumnos.

## 2. Comenzar y abrir proyectos

1. Elegí **Archivo → Nuevo proyecto** y seleccioná los componentes y el paquete de propiedades.
2. O bien usá **Archivo → Abrir proyecto** para recuperar un JSON guardado.
3. Otra opción es **Ejemplos → 01…13**. Cargar un ejemplo sustituye el proyecto activo después de la confirmación correspondiente: primero guardá los cambios.
4. Usá **Archivo → Guardar proyecto** para descargar los datos del diagrama, componentes y especificaciones. Guardar **no equivale a exportar los resultados calculados**.

La aplicación no calcula automáticamente cuando abrís un proyecto. Si cambiás condiciones físicas, los resultados anteriores dejan de representar el estado actual y deben recalcularse.

## 3. Diagrama, equipos, corrientes y conexiones

La **biblioteca Equipos y corrientes** contiene los símbolos disponibles. Se pueden agregar al diagrama con clic o arrastre, mover los símbolos y conectar los extremos de las corrientes con los puertos del equipo. Cada conexión debe enlazar una corriente y una operación, no dos equipos directamente.

Seleccioná una corriente o un equipo para editar sus datos en **Propiedades**: caudal, temperatura, presión, composición, paquete o parámetros del equipo según corresponda. Los enlaces bajo cada objeto apuntan a los capítulos técnicos de esta wiki.

Un dibujo coherente es condición necesaria, pero no suficiente, para obtener un cálculo físicamente válido. Consultá [Diagramas y conexiones](diagramas-conexiones.md) y [Grados de libertad](grados-libertad-especificaciones.md).

## 4. Organizar el espacio de trabajo

En pantallas de escritorio, **Equipos y corrientes** y **Propiedades** pueden cambiar de lado y abrirse como paneles flotantes dentro de la página. Cuando un panel está flotante, arrastrá su **cabecera** para desplazarlo y utilizá **Acoplar** para regresarlo al editor. El menú **Ver → Restablecer distribución** devuelve la disposición estándar. La distribución se conserva localmente en ese navegador, no dentro del JSON del proceso.

Los bordes laterales permiten ajustar el ancho de los paneles acoplados. **Corrientes y balances** puede separarse en otra ventana sincronizada mediante su icono **↗**, cambiar de altura arrastrando el borde superior, ocultarse y recuperarse desde **Ver**.

El **diagrama** puede ampliarse mediante **Solo diagrama**, separarse en otra ventana sincronizada mediante **Ver → Separar diagrama** y mostrarse en pantalla completa. Las ventanas separadas requieren que el navegador permita ventanas emergentes. Los paneles flotantes internos no son ventanas independientes del sistema operativo.

En teléfonos o pantallas pequeñas se prioriza la disposición acoplada y adaptable: algunas funciones de flotación se desactivan para evitar superposiciones.

## 5. Modelos termodinámicos, composición y unidades

Elegí un modelo apropiado antes de calcular. Por ejemplo, **IAPWS-IF97** es adecuado para agua pura dentro de sus límites; mezclas como agua–etanol pueden requerir modelos de coeficientes de actividad y parámetros aplicables. Consultá [Modelos termodinámicos](modelos-termodinamicos.md).

Desde **Herramientas → Configuración** se puede cambiar la representación de unidades, manteniendo las condiciones físicas internas. La calculadora de unidades y composición ayuda a convertir caudales y fracciones. Distinguir base **másica** de **molar**, y presión **absoluta** de **manométrica**. Las estimaciones de gas ideal y presión parcial no sustituyen un cálculo de equilibrio.

Ver [Unidades y bases](unidades-bases-conversiones.md).

## 6. Comprobar y calcular

1. Completá las condiciones de alimentación y los parámetros de cada operación.
2. Usá **Simulación → Revisar especificaciones y conexiones**. Corregí advertencias sobre puertos, datos incompletos o límites de operaciones.
3. En el pie, seleccioná explícitamente el motor **Windows** o **Debian** y comprobá su disponibilidad. **No hay conmutación automática** si uno falla.
4. Pulsá **Calcular**. La solicitud llega al gateway y luego al motor DWSIM seleccionado.
5. Consultá la tabla de **Corrientes y balances**, el modelo, las temperaturas, presiones, caudales y los residuos de cada equipo.

Si un motor no está disponible, todavía podés editar y guardar el proyecto, pero **no inventar resultados**. Un cálculo completado tampoco garantiza por sí solo que se eligió un buen modelo físico.

Ver [Diagnóstico y validación](diagnostico-validacion.md).

## 7. Exportar, comparar y documentar

Desde **Archivo** exportá corrientes o balances a **CSV** o el libro **Excel**. Revisá las unidades de las columnas, porque las descargas pueden utilizar unidades canónicas diferentes de la representación elegida para la pantalla.

Para una práctica reproducible conservá el proyecto JSON, la fecha, el motor seleccionado, las condiciones, el paquete termodinámico, los resultados y las advertencias. Se recomienda contrastar balances de materia y energía por componente y por equipo. Ver [Reproducibilidad e informes](reproducibilidad-informes.md).

## 8. Ejercicios y ayuda contextual

Los 13 ejemplos sirven como casos de estudio: mezcla, operaciones térmicas, divisor, mezclas multicomponentes, reactores, flash, bomba/válvula e intercambiador. La guía de ejercicios del portal contiene los resultados documentados por caso; se accede desde **Ejemplos** y permite volver al editor sin sustituir el proyecto.

El **Ejemplo 13** incluye una actividad de predicción y comparación de temperaturas que se inicia desde el inspector del intercambiador cuando está cargado el ejemplo. Sus valores históricos se distinguen de los resultados nuevos del motor.

Ruta sugerida para estudiar: [Fundamentos de corrientes](fundamentos-corrientes-balances.md) → [Energía](energia-operaciones-termicas.md) → [Bombas y válvulas](bombas-valvulas-presion.md) → [Flash](equilibrio-fases-flash.md) → [Prácticas integradoras](practicas-integradoras.md).

## 9. Problemas frecuentes

| Situación | Qué revisar |
|---|---|
| No se puede calcular | Motor seleccionado, conexión, sesión válida y diagnóstico de parámetros |
| Faltan resultados después de un cambio | Recalcular el proyecto; se invalidaron los resultados previos |
| No se ve una ventana separada | Permisos de ventanas emergentes en el navegador |
| Se perdió un panel flotante de vista | **Ver → Restablecer distribución** |
| La composición no cierra | Suma de fracciones, base másica/molar y conversiones |
| El motor responde pero el cálculo falla | Datos de equipo, conexiones, paquete de propiedades y validación |

Para problemas de acceso: [Acceso y clases](acceso.md). Para los límites y la interpretación científica: [Diagnóstico y validación](diagnostico-validacion.md).

## 10. Alcance y fuentes

Los cálculos se realizan en régimen estacionario y actualmente no admiten recirculaciones cerradas en el editor ΣIM. Las columnas de destilación, los PFR/CSTR cinéticos y otros equipos avanzados de DWSIM **no deben presentarse como implementados en este editor**.

**Documentación fuente:** [Repositorio ΣIM](https://github.com/lemeit/lemeit-sim) · [DWSIM: tutoriales oficiales](https://dwsim.org/tutorials/en/index.html). Esta guía es documentación propia de ΣIM, no una traducción oficial de DWSIM.

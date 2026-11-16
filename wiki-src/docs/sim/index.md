# ΣIM · Simulación de procesos

[![sitio: sim.lemeit.ar](https://img.shields.io/badge/sitio-sim.lemeit.ar-009688?style=flat-square&labelColor=555555)](https://sim.lemeit.ar/)
[![código: lemeit-sim](https://img.shields.io/badge/c%C3%B3digo-lemeit--sim-009688?style=flat-square&labelColor=555555)](https://github.com/lemeit/lemeit-sim)

Simulador educativo de procesos con interfaz web en español y motor DWSIM 10.2.15. Portal documentado: **0.17.2**, revisión del **16 de noviembre de 2026**. La aplicación está publicada y continúa en desarrollo; no reproduce todo el editor de escritorio de DWSIM.

## Accesos y documentación

- [Ingresar al simulador](https://sim.lemeit.ar/acceso.html).
- [Acceso por clase y administración](acceso.md).
- [Guía de uso](https://sim.lemeit.ar/ayuda.html), [13 ejemplos](https://sim.lemeit.ar/ejemplos.html) y [catálogo](https://sim.lemeit.ar/catalogo.html). Los recursos del portal requieren una sesión válida.
- [Índice técnico](https://github.com/lemeit/lemeit-sim/blob/main/docs/README.md), [historial](https://github.com/lemeit/lemeit-sim/blob/main/docs/CHANGELOG.md) y [comprobaciones](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md).
- [Ficha docente](https://profe.lemeit.ar/projects/sim/).

## Ingreso y clases

El alumno ingresa con un código temporal de 12 caracteres, sin correo ni cuenta individual. El pie muestra su clase, acceso restante y salida. El administrador usa una contraseña independiente y un panel privado para crear, consultar y desactivar accesos; no muestra contador.

Los nuevos códigos de clases activas se pueden volver a copiar desde el panel, incluso después de recargar. Los anteriores a 0.17.0 siguen funcionando, pero no pueden recuperarse si no se conservaron. El acceso y los permisos se validan en servidor: cambiar una opción del formulario no concede rol administrativo.

**Estado de motores** es una consulta de disponibilidad para usuarios autenticados; **Administración de clases** requiere administrador. No hay botones remotos para encender o reiniciar equipos. Ver [guía de acceso](acceso.md).

## Fundamentos para aprender simulación

Comenzamos una serie de guías propias en español basada en conceptos de ingeniería química y vinculada a prácticas reproducibles del simulador. La serie comienza con [**Corrientes, estado termodinámico y balances**](fundamentos-corrientes-balances.md), y continúa con [**Grados de libertad y especificaciones**](grados-libertad-especificaciones.md), para distinguir datos independientes de resultados, elegir pares flash y reconocer errores de subespecificación y sobreespecificación. Cada capítulo contiene ejercicios en ΣIM y referencias a los tutoriales oficiales de DWSIM. No se presenta como traducción oficial.

## Qué permite estudiar

Corrientes de materia y energía; mezcladores, divisores, calentadores y enfriadores; separador flash vapor–líquido; bomba y válvula; intercambiador de dos corrientes; reactores de conversión y equilibrio dentro del alcance documentado. El selector termodinámico y la composición se configuran por proyecto.

Hay 13 ejemplos con datos, resolución, referencias y límites, además de conversión de unidades, bases másica/molar, exportaciones CSV/Excel y herramientas de revisión. La guía integrada permite volver al mismo proyecto. Los diálogos se cierran por botón, Escape o clic fuera.

El alcance es estacionario, sin recirculaciones. CSTR/PFR cinéticos, columnas y otras operaciones del motor requieren adaptación y validación. Los resultados de los ejemplos son modelos calculados, no mediciones experimentales ni validación física independiente.

## Recorrido de trabajo

1. Ingresar con el código de clase y comprobar la identidad en el pie.
2. Crear un proyecto con compuestos/modelo o abrir explícitamente un ejemplo.
3. Agregar equipos y corrientes, conectar puertos y completar condiciones.
4. Seleccionar manualmente Windows o Debian, comprobar disponibilidad y calcular.
5. Revisar mensajes, corrientes y balances; guardar el JSON y exportar resultados si hace falta.

No hay cambio automático de motor. Un servicio caído no impide preparar o guardar el proyecto con una sesión válida, pero no se generan resultados sin cálculo. Guardar antes de salir o volver a ingresar.

## Arquitectura, validación y publicación

El navegador edita el diagrama; el Worker de Cloudflare valida sesión/rol y reenvía solicitudes a gateways privados de Windows o Debian. DWSIM se ejecuta fuera del navegador. Las computadoras, gateways y túneles deben estar disponibles; publicar el portal no inicia esos procesos.

Las comprobaciones de acceso e interfaz y los recorridos locales con base en memoria no equivalen a nuevas simulaciones reales. Los informes de cálculos conservan fecha y motor; el Ejemplo 13 tiene comparación Windows–Debian documentada en 0.14.0. No se afirma prueba de carga simultánea de una clase ni validación experimental.

El repositorio es `lemeit/lemeit-sim`. Workers Builds publica desde su configuración; la versión efectivamente cargada se consulta en **Ayuda → Acerca de** y se contrasta con el despliegue exitoso, no solo con una fusión en GitHub. La wiki no configura secretos ni pone en marcha motores.

## Identidad y licencias

Nombre exacto: **ΣIM**, sigma griega mayúscula seguida de I y M latinas. Bajada: **Simulación de procesos**. Conserva dominio y repositorio del nombre inicial SIMPROX. La aplicación mantiene su diseño técnico negro/blanco/rojo; la wiki conserva su propia identidad.

El proyecto usa DWSIM y declara GPL-3.0-or-later; consultar atribuciones y condiciones en el repositorio. No es un producto oficial de DWSIM. Es distinto de [Bestiario · Seres Térmicos](https://profe.lemeit.ar/projects/simus/bestiario/).

# ΣIM · Simulación de procesos

[![sitio: sim.lemeit.ar](https://img.shields.io/badge/sitio-sim.lemeit.ar-009688?style=flat-square&labelColor=555555)](https://sim.lemeit.ar/)
[![código: lemeit-sim](https://img.shields.io/badge/c%C3%B3digo-lemeit--sim-009688?style=flat-square&labelColor=555555)](https://github.com/lemeit/lemeit-sim)

Simulador educativo de procesos con interfaz web en español y motor DWSIM 10.2.15. Versión documentada en esta wiki: **0.53.3**; comprobar la versión desplegada en Ayuda → Acerca de. La aplicación está publicada y continúa en desarrollo; no reproduce todo el editor de escritorio de DWSIM.

## Accesos y documentación

- [Ingresar al simulador](https://sim.lemeit.ar/acceso.html).
- [Acceso por clase y administración](acceso.md).
- [Manual de uso de ΣIM](manual-de-uso.md) (esta wiki), [20 ejemplos del portal](https://sim.lemeit.ar/ejemplos.html) y [catálogo de equipos](https://sim.lemeit.ar/catalogo.html). Las páginas del portal requieren una sesión válida.
- [Índice técnico](https://github.com/lemeit/lemeit-sim/blob/main/docs/README.md), [historial](https://github.com/lemeit/lemeit-sim/blob/main/docs/CHANGELOG.md) y [comprobaciones](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md).
- [Ficha docente](https://profe.lemeit.ar/projects/sim/).

## Ingreso y clases

El alumno ingresa con un código temporal de 12 caracteres, sin correo ni cuenta individual. El pie muestra su clase, acceso restante y salida. El administrador usa una contraseña independiente y un panel privado para crear, consultar y desactivar accesos; no muestra contador.

Los nuevos códigos de clases activas se pueden volver a copiar desde el panel, incluso después de recargar. Los anteriores a 0.17.0 siguen funcionando, pero no pueden recuperarse si no se conservaron. El acceso y los permisos se validan en servidor: cambiar una opción del formulario no concede rol administrativo.

**Estado de motores** es una consulta de disponibilidad para usuarios autenticados; **Administración de clases** requiere administrador. No hay botones remotos para encender o reiniciar equipos. Ver [guía de acceso](acceso.md).

## Novedades de la interfaz hasta 0.53.3

- Menús y barra por categorías (Archivo, Proyecto, Diagrama, Simulación y Vista), inserción por equipos y corrientes y paleta lateral.
- Diagramas con desplazamiento mediante Espacio + arrastre o rueda central; puertos de conexión visibles.
- Resultados separados en vistas de balances y corrientes, con paneles acoplables, flotantes o en ventana independiente.
- ▶ Calcular y **Explorar proceso…**: navegación didáctica de equipos y corrientes con ecuaciones, valores y balances. No vuelve a ejecutar DWSIM.
- Ficha de proyecto, actividad y anotaciones editables en el diagrama, reportes PDF A4/A3/Oficio y guardado de proyectos en nube por alumno.
- La distinción entre potencia térmica intercambiada y cierre energético está integrada desde 0.47.5.
- Barra reordenable, controles +/−, tema claro/oscuro común y separadores mínimos entre paneles apilados.
- Catálogo de compuestos con favoritos, búsqueda completa y Mostrar más; explorador de proceso con altura estable.
- Ejemplo 20: dinámica de tanque con DWSIM comprobada en Windows y Debian (121 muestras por motor). Gráficas admite dos curvas, un segundo eje Y para unidades distintas, formato por serie y exportación SVG, PNG, JPG y CSV. Ver [manual de uso](manual-de-uso.md).

- Desktop 0.53.3-alpha.1 guarda copias `.sim` desde el selector del sistema. Online conserva la biblioteca en nube y la descarga JSON.
- La barra concentra **Calcular** junto a los controles de simulación. Las viñetas de exploración se pueden anclar al diagrama, mover y redimensionar.

## Fundamentos para aprender simulación

Comenzamos una serie de guías propias en español basada en conceptos de ingeniería química y vinculada a prácticas reproducibles del simulador. La serie comienza con [**Corrientes, estado termodinámico y balances**](fundamentos-corrientes-balances.md), y continúa con [**Grados de libertad y especificaciones**](grados-libertad-especificaciones.md), para distinguir datos independientes de resultados, elegir pares flash y reconocer errores de subespecificación y sobreespecificación. El [**capítulo 3: selección de modelos termodinámicos**](modelos-termodinamicos.md) compara IAPWS-IF97, NRTL, UNIQUAC, UNIFAC, Peng–Robinson y SRK con criterios de validez y prácticas. Cada capítulo contiene ejercicios en ΣIM y referencias a los tutoriales oficiales de DWSIM. No se presenta como traducción oficial.

## Itinerario completo de fundamentos

| Capítulo | Tema | Práctica relacionada |
|---|---|---|
| [01](fundamentos-corrientes-balances.md) | Corrientes, estado y balances | Mezclador |
| [02](grados-libertad-especificaciones.md) | Grados de libertad y especificaciones | Divisor y calentador |
| [03](modelos-termodinamicos.md) | Selección termodinámica | Agua y agua–etanol |
| [04](equilibrio-fases-flash.md) | Equilibrio de fases y flash | Ejemplo 11 |
| [05](energia-operaciones-termicas.md) | Energía y operaciones térmicas | Ejemplos 01–03, 13 |
| [06](bombas-valvulas-presion.md) | Bombas, válvulas y presión | Ejemplo 12 |
| [07](reactores-estequiometria.md) | Reactores y estequiometría | Ejemplos 07–08 |
| [08](diagramas-conexiones.md) | Diagramas y conexiones | Mezclador y divisor |
| [09](unidades-bases-conversiones.md) | Unidades y composición | Agua–etanol |
| [10](diagnostico-validacion.md) | Diagnóstico y validación | Auditoría del flash |
| [11](practicas-integradoras.md) | Taller integrador y rúbrica | Biblioteca de ejemplos |

## Nivel intermedio: profundización de ingeniería

La serie continúa con cinco capítulos **conceptuales y didácticos**. Varios temas requieren DWSIM de escritorio y aún no están implementados en el editor ΣIM: no se confunde la explicación teórica con una función disponible.

| Capítulo | Contenido | Modalidad |
|---|---|---|
| [12](recirculaciones-convergencia.md) | Recirculaciones, corriente de corte y convergencia | Balance, iteración y ensayos de recirculación; validación física de los lazos aún en curso |
| [13](destilacion-etapas-equilibrio.md) | Destilación y balances de columna | Balance binario resuelto; columna fuera de ΣIM |
| [14](cinetica-dimensionamiento-reactores.md) | Cinética, CSTR y PFR | Dimensionamiento analítico; fuera de ΣIM |
| [15](sensibilidad-escenarios.md) | Derivadas, escenarios y comparación de modelos | Variaciones manuales de ejemplos disponibles |
| [16](reproducibilidad-informes.md) | Trazabilidad, evidencia y rúbrica de informes | Aplicable a los ejemplos disponibles |

**Recorrido sugerido:** estudiar los capítulos 01–11, resolver los ejercicios de 12–14 de forma analítica y usar 15–16 para documentar prácticas reproducibles. Ningún ejemplo didáctico nuevo de esta ampliación se presenta como ejecución nueva del motor.

Los once capítulos forman una **primera serie introductoria completa**, no un manual exhaustivo del DWSIM de escritorio. Los ejercicios distinguen cálculos analíticos nuevos de resultados DWSIM documentados previamente.

## Qué permite estudiar

Corrientes de materia y energía; mezcladores, divisores, calentadores y enfriadores; separador flash vapor–líquido; bomba y válvula; intercambiador de dos corrientes; reactores de conversión y equilibrio dentro del alcance documentado. El selector termodinámico y la composición se configuran por proyecto.

La biblioteca contiene casos introductorios y otros de recirculación en desarrollo; algunos requieren validación numérica adicional. Además dispone de conversión de unidades, bases másica/molar, exportaciones CSV/Excel y herramientas de revisión. La guía integrada permite volver al mismo proyecto. Los diálogos se cierran por botón, Escape o clic fuera.

El cálculo general es estacionario, con un módulo de dinámica de tanque específico en desarrollo. El gateway admite un primer modo de recirculación simple, con validación cruzada y convergencia aún pendientes para los ejercicios 17–19; no representa todos los métodos de DWSIM. CSTR/PFR cinéticos, columnas y otras operaciones del motor requieren adaptación y validación. Los resultados de los ejemplos son modelos calculados, no mediciones experimentales ni validación física independiente.

## Recorrido de trabajo

La referencia operativa para los controles de la aplicación, la organización de paneles y las exportaciones es el [manual de uso del simulador](manual-de-uso.md). Para temas científicos se utilizan los capítulos de fundamentos y el nivel intermedio.


1. Ingresar con el código de clase y comprobar la identidad en el pie.
2. Crear un proyecto con compuestos/modelo o abrir explícitamente un ejemplo.
3. Agregar equipos y corrientes, conectar puertos y completar condiciones.
4. Seleccionar manualmente Windows o Debian en el pie, comprobar disponibilidad y pulsar ▶ Calcular.
5. Revisar corrientes y balances, y explorar las ecuaciones del proceso con resultados vigentes.
6. Usar **Guardar en nube** con identidad de alumno y retomar desde **Mis proyectos en la nube**. Para el informe, exportar las tablas, las gráficas o el PDF.

Con motor fijado no hay cambio automático de motor; el modo Automático comprueba disponibilidad antes de calcular. Un servicio caído no impide preparar o guardar el proyecto en nube con una sesión válida, pero no se generan resultados sin cálculo. En la nube quedan el diagrama, las condiciones y el formato de Gráficas. Al abrir un proyecto, podés consultar los resultados de su última corrida DWSIM guardada. Exportá CSV para usar las muestras en una planilla.

## Disponibilidad del cálculo

Elegí Motor principal o Motor alternativo y comprobá su disponibilidad. Si un servicio no está disponible, podés seguir preparando y guardando el proyecto. Consultá los detalles de infraestructura en la documentación técnica del repositorio.


## Identidad y licencias

Nombre exacto: **ΣIM**, sigma griega mayúscula seguida de I y M latinas. Bajada: **Simulación de procesos**. Conserva dominio y repositorio del nombre inicial SIMPROX. La aplicación mantiene su diseño técnico negro/blanco/rojo; la wiki conserva su propia identidad.

El proyecto usa DWSIM y declara GPL-3.0-or-later; consultar atribuciones y condiciones en el repositorio. No es un producto oficial de DWSIM. Es distinto de [Bestiario · Seres Térmicos](https://profe.lemeit.ar/projects/simus/bestiario/).
